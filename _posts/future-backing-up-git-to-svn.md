
```python
#! /usr/bin/python

import os
import shlex
import shutil
import subprocess
import sys
import traceback

# The root path that all repos are backed up to in SVN
SVN_ROOT = "https://example.com/path/to/svn"
# The directory that houses the mirrors locally.
TMP = ".svn-mirrors"

try:
    # Resolve Unix username and group name
    import grp
    import pwd
    def resolveOwnerAndGroup(uid, gid):
        return (pwd.getpwuid(uid)[0], grp.getgrgid(gid)[0])
except ImportError:
    # Fallback support for non-Unix
    def resolveOwnerAndGroup(uid, gid):
        return ("(Not-Unix)", "(Not-Unix)")


def call(cmd):
    """Returns (returnCode, stdout, stderr)"""
    p = subprocess.Popen(shlex.split(cmd), stdout = subprocess.PIPE,
            stderr = subprocess.PIPE)
    out, err = p.communicate()
    r = p.wait()
    return (r, out, err)


def checked(cmd):
    """Raises an Exception if cmd returns non-zero.  Otherwise, returns
    (stdout, stderr).
    """
    retval, out, err = call(cmd)
    if retval != 0:
        raise Exception("Non-zero return from {}\nError code {}\nstdout: {}\n"
                "stderr: {}".format(cmd, retval, out, err))
    return out, err


def processRepo(d):
    """Handles cloning local repo d to SVN.  Note that bare repos should be
    used for sharing locally, but git-svn does not support bare repositories.
    So, we clone to an intermediate repo.
    """
    assert d.endswith('.git')
    print("==== {} ====".format(d))

    # Do we have access?
    if not os.access(d, os.R_OK):
        stat = os.stat(d)
        user, group = resolveOwnerAndGroup(stat.st_uid, stat.st_gid)
        print("Cannot read!  Owner:group is {}:{}".format(user, group))
        return

    svnName = d[:-4]
    svnUrl = "{}/{}".format(SVN_ROOT, svnName)
    r, o, e = call("svn log -l 1 {}".format(svnUrl))
    if r != 0 and 'path not found' in e.lower():
        # Directory does not exist, create it
        print("Creating SVN repository at {}".format(svnUrl))
        checked('svn mkdir -m "Adding {} repository mirror from git" {}'
                .format(d, svnUrl))
    else:
        print("...repo has folder in SVN")

    for f in [ 'trunk', 'branches', 'tags' ]:
        # The trunk folder must work, or git svn will not work.
        url = '{}/{}'.format(svnUrl, f)
        r, o, e = call("svn log -l 1 {}".format(url))
        if r != 0 and 'path not found' in e.lower():
            print("Creating {} directory".format(f))
            checked('svn mkdir -m "Adding {} for {} (from git)" {}'
                    .format(f, svnName, url))

    # Now, the directory exists.  Check if the mirror repository exists, and if
    # not, configure it.
    mirror = os.path.abspath(os.path.join(TMP, svnName))
    if not os.path.lexists(mirror):
        print("Creating local mirror at {}".format(mirror))
        checked("git clone {} {}".format(d, mirror))
    else:
        print("...local mirror exists")

    # Ensure that the svn-remote entry exists in the mirror's config
    svnConf = os.path.join(mirror, ".git", "config")
    if 'svn-remote "svn"' not in open(svnConf).read():
        with open(svnConf, 'a') as f:
            f.write('\n[svn-remote "svn"]\n')
            f.write('    url = {}\n'.format(svnUrl))
            f.write('    fetch = trunk:refs/remotes/svn/trunk\n')
            f.write('    branches = branches/*:refs/remotes/svn/branches/*\n')
            f.write('    tags = tags/*:refs/remotes/svn/tags/*\n')

    # !!! Anything below this point deals with the clone, and this script has
    # been chdir'd into the clone's directory!
    odir = os.getcwd()
    os.chdir(mirror)

    try:
        # Ensure that our idea of what the svn looks like is up to date
        print("...fetching latest svn information")
        checked("git svn fetch svn")

        # And that our local is up-to-date
        print("...fetching latest git information")
        checked("git fetch --all")

        # Find branches that exist in origin
        brLocal = set()
        for maybeRepo in checked("git branch -r")[0].split('\n'):
            maybeRepo = maybeRepo.strip().rsplit(' ', 1)[-1]
            if not maybeRepo:
                # Empty string, don't log
                continue
            elif maybeRepo.startswith("origin/"):
                if maybeRepo not in brLocal:
                    print("...will backup {}".format(maybeRepo))
                    brLocal.add(maybeRepo)
            else:
                print("...will NOT backup {} (not origin)".format(maybeRepo))

        # Find svn branches
        brSvn = (checked("svn ls {}/branches".format(svnUrl))[0].strip()
                .split('\n'))
        brSvn = set(['svn/trunk'] + [ 'svn/branches/' + b.strip().rstrip('/')
                for b in brSvn if b.strip() ])
        print("...found SVN branches: {}".format(brSvn))

        # Maps from SVN names to origin names
        def makeOrigin(br):
            """Maps from SVN name to origin"""
            if br == "svn/trunk":
                return "origin/master"
            elif br.startswith("svn/branches"):
                return "origin/{}".format(br[13:])
            else:
                raise ValueError("Unknown translation for {}".format(br))

        def makeSvn(br):
            """Maps from origin name to SVN name"""
            if br == "origin/master":
                return "svn/trunk"
            elif br.startswith("origin/"):
                return "svn/branches/{}".format(br[7:])
            else:
                raise ValueError("Unknown translation for {}".format(br))

        # Delete SVN branches that no longer exist
        for br in brSvn:
            obr = makeOrigin(br)
            if obr not in brLocal:
                print("Deleting SVN branch {}".format(br))
                checked("svn delete {}/{} -m 'Branch {} no longer exists in "
                        "git'".format(svnUrl, br[4:], br))

        # Backup git branches to SVN
        for br in brLocal:
            sbr = makeSvn(br)
            if sbr not in brSvn:
                # Brand new branch!
                print("Creating SVN branch {}".format(sbr))
                checked("svn mkdir {}/{} -m 'New branch {} from git'".format(
                        svnUrl, sbr[4:], sbr))
                checked("git svn fetch svn")

            # Update the branch by rebasing any new commits onto the SVN
            # version
            print("Cloning branch {} to SVN via svn-rebase branch".format(br))
            checked("git checkout master")
            call("git branch -D svn-rebase")
            checked("git checkout -b svn-rebase {}".format(br))
            checked("git rebase {}".format(sbr))
            checked("git svn dcommit")
            checked("git checkout master")
            call("git branch -D svn-rebase")
    finally:
        os.chdir(odir)


if __name__ == '__main__':
    try:
        os.makedirs(TMP)
    except OSError, e:
        # If it already existed, that's OK
        if e.errno != 17:
            raise

    # First order of business, make sure the git folder exists
    r, o, e = call("svn log -l 1 {}".format(SVN_ROOT))
    if r != 0 and 'path not found' in e.lower():
        # Make the parent directory
        checked('svn mkdir -m "Adding root folder for git repository mirror" '
                '{}'.format(SVN_ROOT))

    for d in os.listdir('.'):
        if d.endswith('.git'):
            try:
                processRepo(d)
            except Exception:
                sys.stderr.write(traceback.format_exc())

```
