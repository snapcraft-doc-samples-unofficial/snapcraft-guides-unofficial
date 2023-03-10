Set up classic confinement for an autotools project
===================================================

Some snaps need to have access to system resources outside the scope allowed by
strict confinement, and are unable to do this via the available interfaces.
These snaps are configured to use classic confinement.

This guide shows how to enable classic confinement for a snap built with the
`autotools plugin`_. The example project used in this guide can be found in `this repository`_.

Change the confinement to classic
---------------------------------

Starting with an existing `snapcraft.yaml` file, change the ``confinement`` setting to ``classic``:

.. code:: yaml

   confinement: classic

This will cause the snap to be built in a way that gives it access to system resources.

Use linters to identify problems
--------------------------------

Snapcraft uses `linters`_ to check for issues during builds.
Linters can only be specified in snaps that use the `core22 base`_. Warnings are still reported for snaps that use older bases.

Run snapcraft to build the snap. This may produce warnings like the following:

.. code:: text

   Lint warnings:
   - classic: usr/bin/autotools-classic-example: ELF interpreter should be set to '/snap/core22/current/lib64/ld-linux-x86-64.so.2'.
   - classic: usr/bin/autotools-classic-example: ELF rpath should be set to '/snap/core22/current/lib/x86_64-linux-gnu'.

If there are many warnings about libraries you can disable the library linter so that only classic linter warnings are shown. See the `linters`_ documentation for details.

Fix linter warnings by patching ELF binaries
--------------------------------------------

The easiest way to handle warnings about the ELF interpreter and rpath is to let Snapcraft automatically patch the binaries using `patchelf`.

This is enabled by default for `core20` classic snaps, and can also be enabled for `core22` classic snaps if you are using Snapcraft 7.3 or a version from the edge channel. Pass the `enable-patchelf` build attribute to the plugin:

.. code:: yaml

   plugin: autotools
   build-attributes:
    - enable-patchelf

This can be removed when automatic patching is enabled for `core22` classic snaps in stable releases.

Fix linter warnings with linker flags
-------------------------------------

In this example, the warnings about the ELF interpreter and rpath can be handled by adding options to the linker:

* ``-Wl,-dynamic-linker=/snap/core22/current/lib64/ld-linux-x86-64.so.2``
* ``-Wl,-rpath=/snap/core22/current/lib/x86_64-linux-gnu``

If the ``LDFLAGS`` environment variable is used by the project, the snapcraft.yaml file can be updated to pass these options to the ``autotools`` plugin, using the ``autotools-configure-parameters`` keyword for projects using the `core20 base`_ or later:

.. literalinclude:: example/snap/snapcraft.yaml
   :language: yaml
   :start-at: plugin: autotools
   :end-at: -Wl,-rpath=/snap/core22/current/lib/x86_64-linux-gnu"

This will only be useful for projects where the `snapcraft.yaml` file is maintained as part of the build system.

Rebuild the snap
----------------

Run Snapcraft again to rebuild the snap, consulting the `Classic linter`_ documentation to resolve further issues.

See also `this article`_ for an overview of the classic linter and a discussion of the issues involved in building snaps for classic confinement.

.. _`this repository`: https://github.com/snapcraft-doc-samples-unofficial/autotools-classic-example
.. _`linters`: https://snapcraft.io/docs/linters
.. _`Classic linter`: https://snapcraft.io/docs/linters-classic
.. _`this article`: https://snapcraft.io/blog/the-new-classic-confinement-in-snaps-even-the-classics-need-a-change
.. include:: /links.rst
