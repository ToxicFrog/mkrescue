This is a collection of useful tools for system recovery and whatnot. It is not intended to be as complete as something like Knoppix, but it's good for taking drive snapshots, testing memory and disks, etc.

It comes with a script, `mkrescue`, which will automatically partition, format, and install syslinux on a flash drive or disk image, then copy these tools and an appropriate syslinux configuration to it.

The "core" contains:
- memtest86+
- x86test
- Hardware Detection Tool
- FreeDOS
- HDAT2

This weighs in at about 3MB, suitable even for tiny USB sticks and flash cards.

As optional modules, you can also install:
- MicroCore Linux 6.1 (10MB)
- TinyCore Linux 6.1 (16MB)
- TinyCore Linux 6.1 + wifi support (61MB)
- CorePlus Linux Installer 6.1 (75MB)
- OpenSUSE Tumbleweed network installer & rescue disk (117MB)
- Puppy Linux "Slacko" 5.7 (161MB)
- Puppy Linux "Tahrpup" 6.0 (199MB)

These modules will download additional components the first time they're installed, so make sure you have a net connection. If you want to pre-download these components for later use, run:

    ./mkrescue -m pre all

and it will download them but do nothing else.

## Usage

    $ ./mkrescue -h
    mkrescue -- make a rescue USB stick.
    Usage: mkrescue [options] modules...
      -i       -- device is an image file, automanage with losetup
      -l label -- give filesystem this label; default "rescue"
      -u       -- update already mounted stick, don't partition/format
      -c       -- tell the modules to "clean" themselves first
      -D       -- debug; print commands being executed
      -m cmd   -- execute command on all modules
      -d dest  -- device or image file to install onto
      modules  -- modules to install, available ones are:

### Quick recipes

Download fresh versions of all optional components without installing anything:

    ./mkrescue -m clean all
    ./mkrescue -m pre all

---
Create an image file that can be dd'd onto a flash drive later:

    dd if=/dev/zero bs=1M count=32 of=disk.img
    ./mkrescue -i -d disk.img

---
Create an all-in-one rescue stick with all modules enabled, using the flash drive /dev/sdb and the label "allinone":

    ./mkrescue -d /dev/sdb -l allinone all

---
Install all of the corelinux modules onto a stick already mounted at /media/rescue:

    ./mkrescue -u -d /media/rescue microcore tinycore wificore coreplus


### Details

    modules

The list of modules to include. If omitted, only the core is installed. To see a list of modules, `ls modules` or `./mkrescue -h`.

    -d destination

Specifies the destination of the install. `destination` should be a device file, which will be partitioned, formatted, and have a new bootloader written. In combination with `-i`, it should instead be a normal file, which will be treated as a whole-disk image; in combination with `-u`, it should be the mount point of the filesystem to copy boot files to.

    -i

Indicates that the argument to `-d` is a whole disk image rather than a device file. The script will automatically use `losetup` to attach and detach it from loop devices as needed.

    -u

Indicates that the argument to `-d` is the path to a mounted filesystem. The script will replace the `/boot/` directory on that filesystem according to its other arguments, but will not edit the partition table, write the bootloader, or create new filesystems. Any changes you have made to `/boot` will be discarded, but the other contents of that directory will be left intact.

    -l label

When writing the FAT filesystem to the device, give it the label `label`. Labels will be uppercased and long ones may be truncated.

Some modules need to know the label in order to write a correct boot configuration, so when using `-u` you should also use `-l` to specify the label of the filesystem being updated.

    -c

Before installing each module, tell it to run its `clean` function. What exactly this means varies from module to module, but usually it involves deleting downloaded files and re-downloading them. To clean all modules without installing anything, use `./mkrescue -m clean all`.

    -m command

Run `command` on the listed modules without doing anything. The only useful ones are likely to be `clean` (clean up downloaded/generated files), `pre` (perform pre-installation setup including downloading necessary components), or `boot` (display boot configuration). See "adding new modules" below for more details.

    -D

Enable debug mode; the script will print every command it executes. This is extremely spammy.


## Adding new modules

Adding small tools can be done directly in `boot/`; anything placed there will be copied to the stick when it's created. Make sure to update `boot/syslinux/syslinux.cfg` accordingly.

To add larger, optional modules, create a subdirectory in `modules/` and put all the module contents there. In that subdirectory, create a file, `.rescue`; this is a shell script that defines various functions used by the control script to set up the module.

At runtime, the functions in this file will have the following environment variables available:

 * `module`: the name of the currently executing module.
 * `label`: the filesystem label of the stick.

They run in a subshell and may freely set or modify the environment without affecting the control script.

The `.rescue` script is expected to define a number of functions which will be called by the control script. It is not *necessary* to define any of these, but without at least `describe` and `boot` the module won't be very useful. For examples of these, look at the existing modules.

Except where noted, the default implementations of these functions do nothing and return 0.

    describe

Outputs a brief description of the module on stdout (with no newline; use `echo -n` or similar). Called to generate help and status messages.

    clean

Deletes downloaded or intermediate files. Called when the user explicitly asks for a clean.

    dest

Outputs a directory name on stdout; the contents of the module will be placed in `/boot/$(dest)/`. Default is to output the module name. The module contents can be copied into `/boot/` itself by having `dest` output `./`.

    pre

Called just before installing the module. This is where code that, for example, downloads necessary files should go.

    boot

Called after installing the module. This function should output, on stdout, the text to be appended to the `/boot/syslinux/syslinux.cfg` file.

    post

Called last. Should do any post-install cleanup necessary other than writing the boot configuration.

## License

The original parts of this project -- the mkrescue script and configuration files -- are licensed under the MIT license; see the file COPYING for the text of the license.

In addition, this package contains the following GPL-licensed software:

- Syslinux (components of)
- Hardware Detection Tool
- memtest86
- x86test
- FreeDOS

These are included unmodified, as a "mere aggregation".

At runtime, it may (depending on user preference) also download components of the following GPL-licensed software:

- Core Linux
- Puppy Linux
- OpenSUSE Linux

It also contains HDAT2, released under a custom freeware license reproduced below:

- You are allowed to use this program on one or more machines at a time.
- You are allowed to distribute this program as long as you do it without profit and without modifications to this license (you may not charge a licensing fee for the program).
- You may redistribute this program included as a support tool for your programs, as long as you notify me by e-mail.
- The software and related documentation are provided "as is", without warranty of any kind.
