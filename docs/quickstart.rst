.. _quickstart:

Quick start
===========

.. _documentation: https://en.wikibooks.org/wiki/QEMU/Networking#SMB_server
.. _GitHub: https://github.com/Cisco-Talos/pyrebox

Fetching PyREBox
----------------

You should always download PyREBox from either the *master* or *dev* branches in GitHub_. Please clone the repository
using *git clone* instead of downloading the zip/tar package, as the git metadata contains important information for
downloading and installing required submodules during the installation process.

Building PyREBox
----------------

Installing dependencies for Debian based distributions: 
::
  apt-get install build-essential zlib1g-dev pkg-config libglib2.0-dev binutils-dev libboost-all-dev autoconf libtool libssl-dev libpixman-1-dev libpython-dev python-pip python-capstone virtualenv

Installing dependencies for CentOS 7:
::
  yum groupinstall 'Development Tools'
  yum install zlib-devel.x86_64 glib2-devel.x86_64 binutils-devel.x86_64 boost-devel.x86_64 autoconf.noarch libtool.x86_64 openssl-devel.x86_64 pixman-devel.x86_64 python-devel.x86_64 libfdt-devel
  yum install epel-release
  yum install python-virtualenv python34-pip.noarch python2-pip.noarch

For RHEL/Fedora:
::
  dnf install make automake gcc gcc-c++ kernel-devel zlib-devel pkgconf-pkg-config glib2-devel binutils-devel boost-devel autoconf libtool openssl-devel pixman-devel python2-devel python2-pip python2-virtualenv capstone-python

Required python packages (see the next paragraph for installation instructions):
::
  ipython>=5,<6 sphinx sphinx-autobuild prettytable pefile capstone distorm3 pycrypto pytz

We strongly recommend to use a virtual env to install your python dependencies. If you have a local installation of volatility, it will intefere with the volatility package used by PyREBox. Create the virtual env:
::
  virtualenv pyrebox_venv

Once it has been created, activate it in order to install your python dependencies:
::
  source pyrebox_venv/bin/activate

To install the python dependencies you can use pip:
::      
  pip install -r requirements.txt

Do not forget to activate your virtual env every time you want to start PyREBox!
::
  source pyrebox_venv/bin/activate

Project configuration and building
::
  ./build.sh


Installing PyREBox
------------------

PyREBox package installation is not yet supported.

Creating a VM image for PyREBox
-------------------------------

At this moment, PyREBox supports any Windows image (32 and 64 bit) that is supported by Volatility.
 
You can create your own image using KVM. In order to avoid compatibility problems, use the pyrebox binaries instead of your system installation qemu binaries:
::
  qemu-img create -f qcow2 -o compat=0.10 images/xpsp3.qcow2 4G
  ./pyrebox-i386 -m 256 -monitor stdio -usb -drive file=images/xpsp3.qcow2,index=0,media=disk,format=qcow2,cache=unsafe -cdrom images/WinXP.iso -boot d -enable-kvm


Proceed with installation, and then boot with network (don't use -net none) and usb support (-usb), and plug in a usb (see Loading a USB image). Let the system install all the drivers
::
  ./pyrebox-i386 -m 256 -monitor stdio -usb -drive file=images/xpsp3.qcow,index=0,media=disk,format=qcow2,cache=unsafe -netdev user,id=network0 -device rtl8139,netdev=network0

Basic QEMU usage
documentation: ----------------

PyREBox is based on QEMU, so in order to start a VM within PyREBox, you need to run it exactly as if you
were booting up a QEMU VM. A couple of example scripts are provided: ``start_i386.sh``, ``start_x86_64.sh``,
you can use them as an example.

The only QEMU monitor option supported currently is *stdio* (``-monitor stdio``).

Some useful QEMU parameters are the following:

Memory, in megabytes
::
  -m 256

Start a prompt on standard input/output in order to interact with the qemu monitor
::
  -monitor stdio

Enable usb support
::
  -usb

If the host(local) mouse pointer isn't properly synchronized with guest(remote) mouse pointer, add
::
  -device usb-tablet

You can specify main image file with unsafe caching. Unsafe caching will make snapshoting much faster
::
  -drive file=images/xpsp3.qcow,index=0,media=disk,format=qcow2,cache=unsafe

Disable networking interfaces. See QEMU documentation for other configuration options
::
  -net none

Start vm at its first snapshot
::
  -loadvm 1

Once you start a VM, you will have a QEMU prompt in which you can run all the QEMU commands, plus those implemented in
PyREBox.

Snapshots
*********

You can load an snapshot when starting a VM by using the -loadvm [snapshot] argument, where [snapshot] is the
snapshot number or descriptor. Snapshots taken when running with KVM are not compatible with snapshots taken
when running the whole system emulation approach (no KVM). So, in order to take a snapshot that can be loaded
with pyrebox, you should not enable KVM for it. Booting up the operating system will be slower, but hopefully
you will only need to do this once.

List snapshots
::
  (qemu)info snapshots

Creating an snapshot
::
  (qemu)savevm init

Loading an snapshot 
::
  (qemu)loadvm init
  (qemu)loadvm 1

Networking
**********

Refer to QEMU documentation. By default, the option ``-net none`` disables networking.

User-mode networking interfaces
::
  -netdev user,id=network0 -device rtl8139,netdev=network0

Loading a usb image (with files)
********************************

Create a usb image template
::
  qemu-img create -f raw usb_image_template.img 256M

Boot QEMU/PyREBox, with usb support ``-usb``, and run the following commands:
::
  (qemu) drive_add 0 if=none,id=stick,file=/path/to/usb_image.img,format=raw
  (qemu) device_add usb-storage,id=stick,drive=stick

On your guest system, partition and format the usb drive. Finally, umount it (safe extract).

Remove the USB drive from QEMU/PyREBox
::
  (qemu) device_del stick 

If you are not sure about which USB drive to remove, you can use the command ``info usb``.

Keep the file, because it can be useful as an empty USB drive template.

Copy the image template (usb_image_template.img) to a new file, and then mount and modify it
::
  mount -o loop,offset=32256 usb_image.img /mnt/location

Copy files to /mnt/location

Unmount
::
  umount /mnt/location

Finally, plug usb image in the machine, and use it!
::
  (qemu)usb_add disk:/path/to/usb/image

Sharing a host directory
************************

Check out existing documentation_ for sharing a host directory with the guest via SAMBA.

Basic PyREBox usage
-------------------

Once you start a VM, you will have a (qemu) prompt in which you can run all the QEMU commands.

PyREBox will first read its configuration file (pyrebox.conf).
::
    [MODULES]
    scripts.script_example.py: True
    scripts.volatility_example: False

    [VOL]
    profile: WinXPSP3x86

    [AGENT]
    name: win_agent_32.exe
    conf: win_agent_32.exe.conf 

    [SYMBOL_CACHE]
    path: symbols.WinXPSP3x86

The [MODULES] section contains a list of python modules (packages and subpackages can be specified using standard python
notation (using dots)). You can enable or disable scripts on demand. These scripts will be automatically loaded.

The [VOL] section contains the volatility configuration. You will need to adjust the profile according to your
operating system version.

The [AGENT] section allows you to configure the name of the agent binary (see documentation related to the agent), 
and the configuration file for that binary.

The [SYMBOL_CACHE] section, allows you to specify the path for a json file that will be used by PyREBox to preserve
resolved symbols between different sessions. This path should be unique for each qemu image you have, and improves
significantly the performance once it is loaded with data on the first execution of the system.

There are PyREBox commands that will allow you to load/unload scripts:

Import a module and initialize it
::
  (qemu) import_module scripts.my_plugin

List loaded modules
::
  (qemu) list_modules

Reload a module, by module handle (you can obtain this handle by listing loaded modules)
::
  (qemu) reload_module 1

Unload a module, by module handle (you can obtain this handle by listing loaded modules)
::
  (qemu) unload_module 1

Start the PyREBox shell
::
  (qemu) sh
