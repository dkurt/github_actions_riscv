# RISC-V "V" (Vector) extension runner

There is one known vendor OS which goes with RVV 0.7.1 ISA out of the box. Download it from Mega: [20211230_LicheeRV_debian_d1_hdmi_8723ds.7z](https://mega.nz/folder/lx4CyZBA#PiFhY7oSVQ3gp2ZZ_AnwYA/folder/xtxkABIB). Below are steps required for OS setup:

## Setup OS

1. Build OpenixCard:

    ```bash
    git clone --recursive --depth 1 https://github.com/YuzukiTsuru/OpenixCard
    sudo apt install cmake build-essential automake autoconf libconfuse-dev pkg-config
    cd OpenixCard
    mkdir build
    cd build
    cmake .. && make -j
    ```

2. Unpack image

    ```bash
    7z e 20211230_LicheeRV_debian_d1_hdmi_8723ds.7z
    cd OpenixCard/build
    ./dist/OpenixCard -d ../../20211230_LicheeRV_debian_d1_hdmi_8723ds.img
    ```

3. Flash `20211230_LicheeRV_debian_d1_hdmi_8723ds.img.dump.out/20211230_LicheeRV_debian_d1_hdmi_8723ds.img` using Rufus of BalenaEtcher to SD card (tested on 64GB SanDisk Ultra)

4. Login by `login:sipeed` and `password:licheepi`

5. Connect to WiFi by GUI through `Preferences -> Connman Settings`. Observed OS hangs at "Assotiation" step but after several tries with board restart it finnaly works. Wired Internet by USB adapter not working.

6. Check vector extension "v":

    ```
    cat /proc/cpuinfo
    processor       : 0
    hart            : 0
    isa             : rv64imafdcvu
    mmu             : sv39
    ```

7. Update apt sources file `/etc/apt/sources.list`:

    ```
    deb http://deb.debian.org/debian sid main
    ```

8. Fix kernel panic: https://wiki.sipeed.com/hardware/en/lichee/RV/user.html#Notes-about-Debian

9. Resize filesystem to SD card (tested on 64GB):

    ```
    $ df -h /
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/root       3.6G  2.8G  636M  82% /

    $ sudo fdisk -l
    The backup GPT table is not on the end of the device.
    Disk /dev/mmcblk0: 59.48 GiB, 63864569856 bytes, 124735488 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 189CD837-ED7A-4651-8372-D86D46652AEC

    Device         Start      End  Sectors  Size Type
    /dev/mmcblk0p1 34336    42399     8064  3.9M Linux filesystem
    /dev/mmcblk0p2 42400    42903      504  252K Linux filesystem
    /dev/mmcblk0p3 42904    43407      504  252K Linux filesystem
    /dev/mmcblk0p4 43408    65543    22136 10.8M Linux filesystem
    /dev/mmcblk0p5 65544    66551     1008  504K Linux filesystem
    /dev/mmcblk0p6 66552    94775    28224 13.8M Linux filesystem
    /dev/mmcblk0p7 94776 16871991 16777216    8G Linux filesystem

    $ sudo fdisk /dev/mmcblk0
    Command (m for help): d
    Partition number (1-7, default 7): 7
    Command (m for help): n
    Partition number (7-128, default 7): 7
    First sector (94776-124735454, default 96256): 94776
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (94776-124735454, default 124735454): 124735454

    Created a new partition 7 of type 'Linux filesystem' and of size 59.4 GiB.
    Partition #7 contains a ext4 signature.

    Do you want to remove the signature? [Y]es/[N]o: y
    Command (m for help): w

    $ sudo reboot
    $ sudo resize2fs /dev/mmcblk0p7
    $ df -h /
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/root        59G  2.8G   54G   5% /
    ```

## Runner setup

* Because of `fence.tso` Illegal Instruction (hardware bug) use .NET build compiled with GCC from https://github.com/dkurt/dotnet_riscv/releases
* Update glibc before usage:

    ```
    sudo apt-get update
    sudo apt-get install -y libc6 libstdc++6
    ```
