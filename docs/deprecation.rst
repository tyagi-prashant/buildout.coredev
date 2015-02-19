Python Deprecation Done Right
=============================

.. warning::

    This document is work in progress.

    Comments appreciated (at plone developer mailing list please)

At some point we

- need to get rid of old code,
- want to unify api style (consistent api),
- fix typos in namings,
- move code around (inside package or to another package).

This document describes best practices to be used in Plone Core.

Help Programmers, No annoyance
------------------------------

Deprecation has to support the consumers of the code - the programmers using it.
From their point of view Plone core code is an API to them.
Any change is annoying to them anyway, but they feel better if deprecation warnings are telling them what to do.

Deprecation always logs at level *warning* and answers the questions:

**"Why is the code gone from the old place? What to do instead?"**

A short message is enough., i.e.:

- "Replaced by new API xyz, found at abc.cde".,
- "Moved to xyz, because of abc.",
- "Name had a typo, new name is "xyz".

All logging has to be done once, i.e. on first usage or first import.
It must not flood the logs.

Use Cases
---------

Renaming
    We may want to rename classes, methods, functions or global or class variables in order to get a more consistent api or because of a typo, etc.
    We never just rename, we always provide a deprecated version logging a verbose deprecation warning with information where to
    import from in future.


Moving a module, class, function, etc to another place
    For some reason, i.e. merging packages, consistent api or resolving cirular import problems, we need to move code around.
    When imported from the old place it logs a verbose deprecation warning with information where to import from in future.

Deprecation of a whole package
    A whole package (folder with ``__init__.py``)

    - all imports still working, logging deprecation warnings on first import
    - ZCML still exists, but is empty (or includes the zcml from the new place if theres no auto import (i.e. for meta.zcml).
    - Not sure if we can raise/log deprecation warnings in this case

Deprecation of a whole python egg
    We will provide a last major release with no 'real' code, only backward compatible (bbb) imports of public API are provided.
    This will be done the way described above for a whole package.
    The README clearly states why it was moved and where to find the code now.


Python and Deprecation
----------------------

Python standard library offers a unified way for deprecation warnings in its `warnings <https://docs.python.org/2/library/warnings.html>`_ module.

.. code-block:: python

    import warnings

    warnings.warn(
        "this is deprecated, better use ...",
        DeprecationWarning
    )

**stderr**: Warnings are written to ``stderr`` by default.
But ``DeprecationWarning`` output is surpressed by default.
Output can be enabled by starting the Python interpreter with the `-W [all|module|once] <https://docs.python.org/2/using/cmdline.html#cmdoption-W>`_ option. It is possible to enable output in code too:

.. code-block:: python

    import warnings
    warnings.simplefilter("module")

**Logging**: Once output is enabled it is possible to `redirect warnings to the logger <https://docs.python.org/2/library/logging.html#logging.captureWarnings>`_:

.. code-block:: python

    import logging
    logging.captureWarnings(True)

Zope 2 and Deprecation
----------------------

.. warning::

    There is a bug with Zope < 2.13.23 together with Python 2.7+ preventing ``DeprecationWarning`` showing up.

In ``zope.conf`` custom filters for warnings can be defined.

.. code-block:: xml

    ...
    <warnfilter>
        action always
        category exceptions.DeprecationWarning
    </warnfilter>
    ...

Using `plone.recipe.zope2instane <https://pypi.python.org/pypi/plone.recipe.zope2instance>`_ this can be generated using the recipe option ``deprecation-warnings = on``.


Helper Packages
---------------

**ATTENTION: BELOW HERE WIP**

Packages `zope.deprecation <https://pypi.python.org/pypi/zope.deprecation/>`_ and `zope.deferredimport <https://pypi.python.org/pypi/zope.deferredimport/>`_ are offering most of the needed functionality.

There is also good documentation for both of this packages for those who want to dive deeper into the topic.
Anyway, here we document the recommeded usage for Plone in a recipe like style.

Examples
--------

Renaming a module
~~~~~~~~~~~~~~~~~

Given we have a Python file at ``plone/foo/bar.py`` and we renamed it to ``plone/foo/baz.py``.
All ``from plone.foo.bar import something`` would be broken.
For all public API inside bar we want to provide a deprecated import.
To enable the import with a deprecation message add to ``plone/foo/__init__.py`` the following code:

.. code-block:: python

    import zope.deferredimport
    zope.deferredimport.initialize()

    zope.deferredimport.deprecated(
        "Import from plone.foo.baz instead",
        something='plone.foo:baz.something',
        someotherthing='plone.foo:baz.someotherthing',
    )


Deprecate a name in a module:


from zope.deprecation.deprecation import deprecated
from zope.deprecation.deprecation import deprecate
from zope.deprecation.deprecation import moved
