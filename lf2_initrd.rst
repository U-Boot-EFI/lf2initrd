Configuring bootloaders Initial RAM disk loading in the Linux EFI stub
======================================================================

On Linux >= 5.7 we have an alternative way to load an initrd, using
EFI_LOAD_FILE2_PROTOCOL, which has become the default if the firmware
provides it.


Reasoning
---------
The EFI spec provides a way of loading multiple EFI executables using the
EFI boot manager. No major distro is using that though and they are all still
using a combination of GRUB+SHIM.  They also use GRUB's config files to couple
specific kernel versions with an appropriate initrd.  Vertical OS'es,  on
smaller embedded systems,  might want to avoid using GRUB for their own reasons.

If they do, they don't have an established way of tightly coupling the
kernel version with a preferred initrd.  There are a few options available,
but those are either limiting or architecture specific.

#. Define initrd= in each Boot#### load option
    * Limits the filesystem they can place the initrd.  The only option they
      have is loading a file from the same filesystem as the kernel image.
    * Newer architectures (e.g RISC-V) don't even have support for the 'initrd='
      command line option.

#. Fixup the chosen node linux,initrd-start and linux,initrd-end properties
    * Need to know beforehand the memory layout and were the kernel expects the
      initrd
    * Architecture specific
    * Kernel version specific
    * initrd is loaded ahead of time

Architecture independent proposal
---------------------------------
The proposed solution combines EFI_LOAD_OPTION and EFI_LOAD_FILE2_PROTOCOL.

Since Linux 5.7 the EFI stub looks for a handle with the device path

::

    /VenMedia(5568e427-68fc-4f3d-ac74-ca555231cc68)

If the protocol is installed, the kernel will make a request with size=0 and an
end node as the device path.  The exact behavior of EFI_LOAD_FILE2_PROTOCOL is
described in § 13.2 Load File 2 Protocol.
Since the initrd filepath is not provided by the kernel (which can't know what
initrd he wishes to load),  it's the firmware's responsibility to provide the
appropriate file,  similar to GRUB.

The EFI_LOAD_OPTION descriptor of Boot#### parameters can provide that path.
The EFI spec § 3.1.3 Load Options specifies for it's FilePathList[].

*A packed array of UEFI device paths. The first element of the array is a
device path that describes the device and location of the Image for this load
option. The FilePathList[0] is specific to the device type. Other device paths
may optionally exist in the FilePathList, but their usage is OSV specific. Each
element in the array is variable length, and ends at the device path end
structure.  Because the size of Description is arbitrary, this data structure
is not guaranteed to be aligned on a natural boundary. This data structure may
have to be copied to an aligned natural boundary before it is used.*

Since the FilePathList[1-N] is OS specific we can define a range of N were the
user can add his initrd(s) path(s) when defining the Boot#### options.

Once the firmware is ready to boot the volatile BootCurrent variable will point
to the Boot#### option that was used.
The EFI Bootloader may scan the device paths of the corresponding  Boot####
variable.  If a file is present, the bootloader may install the necessary
protocol.

Once (and if) the kernel efi-stub discovers the protocol and tries to load an
initrd, the EFI firmware must locate the file(s) via the device paths in the
EFI_LOAD_OPTION of the correct Boot#### variable and copy the buffer into the 
kernel provided memory.
