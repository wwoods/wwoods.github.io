---
layout: post
title: Cython and Default Directives
tags: cython python
eye_catch: /assets/img/logo-cython.svg
---

[Cython](http://cython.org) has a number of very useful [compiler directives](http://docs.cython.org/src/reference/compilation.html#compiler-directives) that modify the compiler's default behavior.  These cover behaviors from enabling code profiling to changing the default python object type used to represent a C string.  However, when using `pyximport`, the simplest way to use Cython in a project, these settings have to be configured per file or per code block, not globally.  With a little ingenuity, this restriction can be overcome.

<!--more-->

It requires wrapping one of pyximport's internal functions, but this can be
done by enforcing default values for these directives just before Cython compiles a file.  The following example injects is has the same effect as putting `# cython: embedsignature=True, profile=True` at the top of every file imported via `pyximport`:

```python

# Allow .pyx files to be seamlessly integrated via cython/pyximport with
# default compiler directives.
import functools
import pyximport.pyximport

# Hack pyximport to have default options for profiling and embedding signatures
# in docstrings.
# Anytime pyximport needs to build a file, it ends up calling
# pyximport.pyximport.get_distutils_extension.  This function returns an object
# which has a cython_directives attribute that may be set to a dictionary of
# compiler directives for cython.
_old_get_distutils_extension = pyximport.pyximport.get_distutils_extension
@functools.wraps(_old_get_distutils_extension)
def _get_distutils_extension_new(*args, **kwargs):
    extension_mod, setup_args = _old_get_distutils_extension(*args, **kwargs)

    if not hasattr(extension_mod, 'cython_directives'):
        extension_mod.cython_directives = {}
    extension_mod.cython_directives.setdefault('embedsignature', True)
    extension_mod.cython_directives.setdefault('profile', True)
    return extension_mod, setup_args
pyximport.pyximport.get_distutils_extension = _get_distutils_extension_new

pyximport.install()
```

Note that this will not forcibly recompile unchanged modules with the new options; you will have to `touch` those files to trigger a compilation with the new configuration.

Conveniently, file comments (the traditional way to apply these directives to an entire file) will still override these settings.

