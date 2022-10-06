# My Linux Kernel Development Workflow


## 'Install' the required dependencies
```
$ nix develop
```

or if youre not using flakes...

```
$ nix-shell linux.nix
```

## Build the kernel
```
$ make x86_64_defconfig
$ make defconfig kvm_guest.config
$ scripts/config --set-val DEBUG_INFO y --set-val DEBUG y  --set-val GDB_SCRIPTS y --set-val DEBUG_DRIVER y

$ make
```

## Create root filesystem
```
$ qemu-img create qemu-image.img 1g
$ mkfs.ext2 qemu-image.img
$ mkdir mnt
# mount -o loop qemu-image.img mnt
# debootstrap --arch amd64 jessie mnt
# chroot mnt /bin/sh
## passdw (set password)
## exit
```

## Run the VM with the built kernel
```
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -drive file=qemu-image.img,index=0,media=disk,format=raw -append "root=/dev/sda nokaslr console=ttyS0 earlyprintk=serial" -enable-kvm -nographic
```

## Debugging
1. Add `-s -S` to the qemu parameters
2. `gdb -ex "add-auto-load-safe-path $(pwd)" -ex "file vmlinux" -ex 'target remote 127.0.0.1:1234' -ex 'hbreak start_kernel'`
3. Debug!
