Python Deprecation Done Right
=============================

At some point we

- need to get rid of old code,
- want to unify api style (consistent api),
- fix typos in namings,
- move code around (inside package or to another package).

This document describes best practices to be used in Plone Core.

Help Programmers, No annoyance
------------------------------

Deprecation has to support the consumers (programmers) of the code.
From their point of view Plone core code is an API to them.
Any change is annoying anyway, but the experience can be reduced if deprecation warning helps them.

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
    For some reason, i.e. merging packages, consitstent api or resolving cirular import problems, we need to move things around.
    When imported from the old place it logs a verbose deprecationWarning with information where to import from in future.

Deprecation of a whole package
    A whole package (folder with ``__init__.py``)

    - all imports still working, logging deprecation warnings on first import
    - ZCML still exists, but is empty (or includes the zcml from the newplace if theres no auto import (i.e. for meta.zcml).
    - Not sure if we can raise/log deprecation warnings in this case

Deprecation of a whole python egg
    We will provide a last major release with no 'real' code, only backward compatible (bbb) imports of public API are provided.
    This will be done the way described above for a whole package.
    The README clearly states why it was moved and where to find the code now.
