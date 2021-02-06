Initial RAM disk loading in the Linux EFI stub
==============================================

Booting Linux requires at least a kernel image and a hardware description.

The hardware description can either be supplied via ACPI tables or via
device trees.

Using an initial RAM disk allows to reduce the kernel size by moving most
code to loadable modules. The initial RAM disk may further comprise
configuration files and an initial execution environment.

Methods to load the initial RAM disk
------------------------------------

Due to historic reasons the Linux kernel EFI stub provides multiple methods
to pass an initial RAM disk.

.. The rest is valid for ARM and RISC-V but how about x86?

/chosen node of the device tree
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The initial RAM disk may be passed to the kernel in memory. The location of the
disk image can be indicated in the device tree by adding parameters
"linux,initrd-start" and "linux,initrd-end" to the "/chosen" node of the device
tree.

This method is used by GRUB for the ARM architecture.

initrd= command line parameter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configuring the Linux kernel with

::

    CONFIG_EFI_GENERIC_STUB_INITRD_CMDLINE_LOADER=y

enables to pass a command line parameter initrd= which indicates the file path
of the initial RAM disk relative to the boot partition.

This configuration parameter has been marked as deprecated and is not available
for the RISC-V architecture.

If loading of the initial RAM disk is successful, the "linux,initrd-start" and
"linux,initrd-end" parameters of the "/chosen" node of the device tree are
overwritten.

EFI_LOAD_FILE2_PROTOCOL
~~~~~~~~~~~~~~~~~~~~~~~

The EFI stub looks for a handle with the device path

::

    /VenMedia(5568e427-68fc-4f3d-ac74-ca555231cc68)

exposing the EFI_LOAD_FILE2_PROTOCOL.

The LoadFile() method of the protocol is called with a file device path

::

    /

containing only the end node.

If loading of the initial RAM disk is successful, the "linux,initrd-start" and
"linux,initrd-end" parameters of the "/chosen" node of the device tree are
overwritten.

If the protocol is found, the initrd= command line parameter is ignored even if
loading via the protocol fails.

As of 2021-02-06 support in boot loaders support is as follows:

The EDK II OvmfPkg package provides the protocol with initrd command since
commit f98608ab3f237 ("OvmfPkg/QemuKernelLoaderFsDxe: add support for new Linux
initrd device path").

Patches for GRUB have been submitted as
https://lore.kernel.org/linux-efi/20201025134941.4805-6-ard.biesheuvel@arm.com/t/

An initial implementation in U-Boot was present in v2020.10 but is deactivated
in v2021.01.
