---
layout: post
title: Easy Sphinx Documentation Without the Boilerplate
tags: sphinx python documentation devtools
eye_catch: /assets/img/2016/2016-06-09-sphinx-tease.png
---

[Sphinx](http://sphinx-doc.org) is a great documentation tool for Python projects.  The style is exceedingly legible and works well with the inline documentation style that Python exhibits.  It even works great for [putting together slides using reveal.js](https://github.com/tell-k/sphinxjp.themes.revealjs).  Getting Sphinx set up for a Python module with multiple submodules, however, can be a nuisance.  Most of the solutions that can be found on the internet involve creating companion `.rst` files for the project, which violates the convenience of Python's inline documentation framework.  This post demonstrates how to get around that for the most part, minimizing the needed amount of Sphinx markup and still producing great documentation.

<!--more-->

Suppose you have this directory structure for your module (`/` is the root of the project):

```
/mod/__init__.py
/mod/inner.py
```

Setup
-----

We'll get to the content of our module in a little bit, but for this method to work, first we need a folder where our Sphinx documentation will reside:

```
$ mkdir docs
$ cd docs
$ sphinx-quickstart
```

`sphinx-quickstart` will ask a bunch of different questions about your project; the defaults are fine for everything, but other extensions such as the math packages are easiest to set up by entering `y` at this step.  At this point, you could build your documentation with `make html`.  However, that documentation won't tell us much about `mod`.  That can be fixed, after modifications to `conf.py`, `index.rst`, and `_templates/autosummary/module.rst`.  Below, lines without a `+ ` or `- ` heading are entries in the default file; lines with one of those headings should be added or removed, respectively.

<div class="code-name" title="conf.py" file="false"></div>
```rst
...

# documentation root, use os.path.abspath to make it absolute, like shown here.
- # import os
- # import sys
- #sys.path.insert(0, os.path.abspath('.'))
+ import os
+ import sys
+ sys.path.insert(0, os.path.abspath('..'))

...

# -- General configuration ------------------------------------------------
+ autoclass_content = "both"  # include both class docstring and __init__
+ autodoc_default_options = {
+         # Make sure that any autodoc declarations show the right members
+         "members": True,
+         "inherited-members": True,
+         "private-members": True,
+         "show-inheritance": True,
+ }
+ autosummary_generate = True  # Make _autosummary files and include them
+ napoleon_numpy_docstring = False  # Force consistency, leave only Google
+ napoleon_use_rtype = False  # More legible

...

# extensions coming with Sphinx (named 'sphinx.ext.*') or your custom
# ones.
- extensions = []
+ extensions = [
+         # Need the autodoc and autosummary packages to generate our docs.
+         'sphinx.ext.autodoc',
+         'sphinx.ext.autosummary',
+         # The Napoleon extension allows for nicer argument formatting.
+         'sphinx.ext.napoleon',
+ ]

...

- html_theme = 'alabaster'
+ html_theme = 'sphinx_rtd_theme'
```

<div class="code-name" title="index.rst" file="false"></div>
```rst
Welcome to mod's documentation!
===============================

- Contents:
-
- .. toctree::
-     :maxdepth: 2
+ .. automodule:: mod
+
+     Members
+     =======
```

In `index.rst`, the "Members" heading helps separate your module's docstring from the auto-generated member documentation.

If you have multiple modules to document or want a cover page that is not your API, then rather than an `.. automodule::` directive, use an `.. autosummary::` directive with a `:toctree: _autosummary` setting (in which case you would not need the "Members" heading lines shown above).

{: .note}
Note that the blank line in `index.rst` is important.  Sphinx distringuishes between parameters and content of a directive by the presence of that blank line.


<div class="code-name" file="module.rst" title="_templates/autosummary/module.rst (a new file; you will need to make the directory)"></div>
```rst
{% raw %}{{ fullname }}
{{ underline }}{% endraw %}

.. contents::
    :local:

.. automodule:: {% raw %}{{fullname}}{% endraw %}

    Members
    =======
```

Again, the "Members" heading helps to separate your module docstrings from autodoc's member documentation.  Note that if you ever change `_templates/autosummary/module.rst`, you will need to delete the `_autosummary` directory to regenerate your documentation (it will get re-made with `make html`).

At this point, running `make html` will document your module.  First, we'll populate the module for this setup.


The Module Itself
=================

Almost there, we just need something to document.  This method requires only one thing in each module's docstring: an `autosummary` directive must be used to document submodules.  Otherwise, any submodules will not have their own content pages generated according to our `module.rst` template, and users will not be able to find their documentation.

<div class="code-name" file="__init__.py" title="/mod/__init__.py"></div>
```python
"""This module demonstrates basic Sphinx usage with Python modules.

Submodules
==========

.. autosummary::
    :toctree: _autosummary

    inner
"""

from .inner import add

VERSION = "0.0.1"
"""The version of this module."""
```

<div class="code-name" file="inner.py" title="/mod/inner.py"></div>
```python
"""The inner module that implements :meth:`add`.
"""

def add(a, b):
    """Adds ``a`` to ``b``.

    Args:
        a (any): First argument to add.
        b (type of a): Second argument to add.

    Returns:
        type of a: The result of ``a + b``.
    """
    return a + b

```

{: .note}
Using [Google style docstrings](http://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html) is recommended to make reading the source files themselves easier.

That's it; running `make html` and opening `/docs/_build/html/index.html` will show:

![Demonstration of docs](/assets/img/2016/2016-06-09-sphinx-ex.png)

If something like a [GitHub Page](https://pages.github.com) is desired for your module, an extra Sphinx makefile target can be added.  That process is detailed on [Nikhil's blog post](http://blog.nikhilism.com/2012/08/automatic-github-pages-generation-from.html).

*Update 2021-06-30: Changed `autodoc_default_flags` to `autodoc_default_options` to address deprecated config.*

*Update 2016-08-04: Changed directory name from `sphinx` to the more community-accepted `docs`.*

*Update 2016-07-28: Added `:local:` to the `.. contents::` directive in the module template.  Looks nicer.  Also updated language to remove references to "Members" heading outside of the template.*

*Update 2016-07-14: Including [Napoleon with Google-style docstrings](http://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html) in default recommended configuration.*

