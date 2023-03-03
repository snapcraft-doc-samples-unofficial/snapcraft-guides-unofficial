Set up classic confinement for a Python project
===============================================

.. include:: ../common/classic-intro.rst

This guide shows how to enable classic confinement for a snap built with the
`python plugin`_. The example project used in this guide can be found in
the `example repository`_.

Change the confinement to classic
---------------------------------

Starting with an existing `snapcraft.yaml` file, change the ``confinement`` setting to ``classic``:

.. code:: yaml

   confinement: classic

This will cause the snap to be built in a way that gives it access to system resources.

Patch ctypes to load system libraries
-------------------------------------

If your application uses `ctypes`_ to access system libraries it will need to
be bundled with a patched version of the module. To bundle ``ctypes``, include
the relevant packages in the ``stage-packages`` list of packages. For the
`core22 base`_, the packages will be the following:

.. literalinclude:: example/snap/snapcraft.yaml
   :language: yaml
   :start-at: stage-packages:
   :end-at: - libpython3.10-stdlib

Patching is done by overriding the build step to perform the build as normal,
by running ``snapcraftctl build``, then applying `a patch`_ to the staged module
file with a script:

.. literalinclude:: example/snap/snapcraft.yaml
   :language: yaml
   :start-at: override-build: |
   :end-at: $SNAPCRAFT_PROJECT_DIR/snap/local/patch-ctypes.sh

An `example script and patch`_ can be found in the `example repository`_.

Run Snapcraft to build the snap with these changes. This may produce warnings
about run-time library paths like the following:

.. code:: text

   Lint warnings:
   - classic: usr/lib/python3.10/lib-dynload/_asyncio.cpython-310-x86_64-linux-gnu.so: ELF rpath should be set to '/snap/core22/current/lib/x86_64-linux-gnu'. (https://snapcraft.io/docs/linters-classic)
   - classic: usr/lib/python3.10/lib-dynload/_bz2.cpython-310-x86_64-linux-gnu.so: ELF rpath should be set to '/snap/core22/current/lib/x86_64-linux-gnu'. (https://snapcraft.io/docs/linters-classic)

These warnings tell us that the run-time library paths of some core Python
modules need to be patched to refer to libraries in the base snap.

Fix linter warnings by patching ELF binaries
--------------------------------------------

The easiest way to handle warnings about the ELF interpreter and rpath is to let Snapcraft automatically patch the binaries using ``patchelf``.

This is enabled by default for ``core20`` classic snaps, and can also be enabled for ``core22`` classic snaps if you are using Snapcraft 7.3 or a version from the edge channel. Pass the ``enable-patchelf`` build attribute to the plugin:

.. literalinclude:: example/snap/snapcraft.yaml
   :language: yaml
   :start-at: build-attributes:
   :end-at: - enable-patchelf

This can be removed when automatic patching is enabled for ``core22`` classic snaps in stable releases.

Rebuild the snap
----------------

Run Snapcraft again to rebuild the snap, consulting the `Classic linter`_ documentation to resolve further issues.

See also `this article`_ for an overview of the classic linter and a discussion of the issues involved in building snaps for classic confinement.

.. _`example repository`: https://github.com/snapcraft-docs/python-ctypes-example
.. _`ctypes`: https://docs.python.org/3/library/ctypes.html
.. _`a patch`: https://github.com/snapcraft-docs/python-ctypes-example/blob/main/snap/local/patches/ctypes_init.diff
.. _`linters`: https://snapcraft.io/docs/linters
.. _`Classic linter`: https://snapcraft.io/docs/linters-classic
.. _`this article`: https://snapcraft.io/blog/the-new-classic-confinement-in-snaps-even-the-classics-need-a-change
.. _`example script and patch`: https://github.com/snapcraft-docs/python-ctypes-example/tree/main/snap/local

.. include:: /links.rst
