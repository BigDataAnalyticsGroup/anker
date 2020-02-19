Linux kernel
============
This extension of linux provides some additional features in linux that facilitates large systems to efficient process in-memory data.

The ```vmcopy()``` implementation provides an efficient way to replicate main-memory data. It can replace an expensive ```malloc() and memcpy()``` operation to copy data virtually instead of creating physical copy. The copy created by ```vmcopy()``` are managed by linux kernel, and is detached from original using copy-on-write (if data is backed by private anonymous memory). The copy retains the properties of original virtual memory area.

Actual linux documentation is available in `Documentation/admin-guide/README.rst`

Commit hash of first commit 48c0dcc537 (needed to create a patch).

#### Build requirements for ubuntu 18.04
`sudo apt-get install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex`

#### syscall specification:
```c
SYSCALL_NR 333

void* vmcopy(void* src, unsigned long length);

// use syscall defined in unistd.h to invoke the system call or,
// use the header file (anker.h) defined in rapido/tests/include/

void *copy = syscall(333, src, length);
```
#### Configure kernel with VMCOPY support
1. Use `make menuconfig` to generate a `.config` file.
2. Enable transparent hugepages by editing `.config` file.
  * Set `CONFIG_TRANSPARENT_HUGEPAGE=y` and `CONFIG_TRANSPARENT_HUGEPAGE_MADVISE=y`.
  * These options are necessary to run the tests specified under rapido/tests/src/
3. Enable `vmcopy()` by setting `CONFIG_ANKER_VMCOPY=y`.
4. Currently `TRANSPARENT_HUGEPAGE` must be enabled to use `vmcopy()`.

#### Testing the kernel
This kernel was developed using the testbed backed by `Qemu`. I have used a trimmed down version of [Rapido](https://github.com/rapido-linux/rapido) that provides a set of scripts to quicky generate `VM Image` and necessary modules using Dracut and boots the image using `Qemu`.

#### Build instructions
1. Generate relevant configure file as mentioned above.
2. make -j
3. INSTALL_MOD_PATH=./mods make modules_install

#### Running tests
1. `cd rapido/tests`
2. `cmake -DCMAKE_CXX_FLAGS=-O2 .`
3. `make -j4`
4. `cd ..`
5. `./cut_anker.sh`
6. Boot the `VM` using `./vm.sh`.
7. Tests are installed in path `/tests/` inside the VM.
8. Use `shutdown` to powerdown the `VM`.

#### Patching linux
1. Create patch using `diff`.
  * `diff -uNr linux.vanilla linux.new > patchfile`
2. Apply patch using `patch`.
  * `cd linux && patch -p1 < ../patchfile`

#### Mount hugetlbfs
1. ```echo 2048 > /proc/sys/vm/nr_hugepages```
2. ```mkdir -p /mnt/hugetlbfs```
3. ```chown -R user:group /mnt/hugetlbfs```
4. ```mount -t hugetlbfs -o uid=<userid>,gid=<groupid>,pagesize=2M,size=4G,nr_inodes=1024 none /mnt/hugetlbfs```

#### License
The code and technology is free for academic use. For commercial use, contact ankur@chainifydb.com
