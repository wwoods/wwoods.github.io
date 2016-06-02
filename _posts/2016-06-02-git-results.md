---
layout: post
title: git-results for Indexing and Organizing Experiments
tags: git-results git science
---

[`git-results`](https://github.com/wwoods/git-results) is a toolkit I've been working on for storing and organizing experiment data.  In my work, I often want to tweak a script's behavior and compare output between different invocations to understand the effects of those tweaks.  The `git-results` extension for `git` provides a means for automatically logging and categorizing these different invocations, providing a memory for which changes were most effective as well as a definitive way to refer to those changes.

<!--more-->

Using `git-results` is documented in the README.  The general usage is something like this:

1. Clone `git-results` and make sure it is on your `PATH`.
2. Set up a git-results.cfg file with the paths that you intend to store results in; for instance:

        ```ini
        [/results]
        build = "python script.py --help"  # Check for SyntaxErrors
        run = "python script.py"
        ```

3. Run `git results results/test -m "Trying out git-results"`.  A message will be given explaining that a new results folder needs to be made and asking for confirmation..
4. Any source changes will be committed and tagged as results/test/1, `python script.py` will be executed, and any new files will be put in results/test/1, as well as stdout and stderr.

Note the `/1` appended to the end of the results directory; `git-results` is designed for multiple invocations of the same results tag.  This is useful both for tolerating errors during runtime as well as for perfecting parameters behind a single idea.  Each experiment invocation will still get its own tag in git, and the change history will be investigable through the tools that git provides.

