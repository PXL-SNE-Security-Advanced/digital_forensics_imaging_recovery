# **Assignment: Disk Imaging and File Recovery in Kali Linux**

## references

- [Kali Linux](https://www.kali.org/)
- [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html)
- [dcfldd](https://dcfldd.sourceforge.io/)
- [foremost](https://tools.kali.org/forensics/foremost)
- [photorec](https://www.cgsecurity.org/wiki/PhotoRec)

---

## **Objective**  

Students will learn how to:

- Add a **1GB virtual disk** to a Kali Linux VM.
- Format, mount, and use the disk.
- Copy and delete a file.
- Create a **forensic disk image** using `dcfldd`.
- Verify the **integrity** of the disk image.
- Recover deleted files using forensic tools.

---

## **Part 1: Adding a New Disk to VMware Kali Linux**

1. **Shut down** the Kali Linux virtual machine.
2. Open **VMware Workstation/Player**.
3. Select the Kali Linux VM and go to **Settings > Add > Hard Disk**.
4. Choose **SCSI (Recommended)** and create a **1GB disk**
5. Start the **Kali Linux VM**.

---

## **Part 2: Identify and Format the New Disk**

### **Step 1: Identify the new disk**

Run the following command in a terminal:

```bash
lsblk
```

```output
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    2G  0 disk 
sdb      8:16   0    1G  0 disk 
sdc      8:32   0 80.1G  0 disk 
└─sdc1   8:33   0 80.1G  0 part /
```

The new disk will likely appear as `/dev/sda` or `/dev/sdb` (verify based on the size).

In the above example, the new disk is `/dev/sdb`.

### **Step 2: Create a Partition Table and Format the Disk**

1. Create a new partition using `fdisk`:

   ```bash
   sudo fdisk /dev/sdb
   ```

   - Press `n` → `p` → `1` → Enter (accept default start and end).
   - Press `p` to print the partition table. (check the partition details)
   - Press `w` to write the changes.

    ```output
    Welcome to fdisk (util-linux 2.39.3).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.
    
    Device does not contain a recognized partition table.
    Created a new DOS (MBR) disklabel with disk identifier 0x31570779.
    Command (m for help): n
    Partition type
    p   primary (0 primary, 0 extended, 4 free)
    e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-2097151, default 2048): 
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151, default 2097151): 

    Created a new partition 1 of type 'Linux' and of size 1023 MiB.

    Command (m for help): p
    Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
    Disk model: VMware Virtual S
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x31570779

    Device     Boot Start     End Sectors  Size Id Type
    /dev/sdb1        2048 2097151 2095104 1023M 83 Linux

    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.
    ```

2. Verify the partition:

   ```bash
   lsblk
   ```

   ```output
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
   sda      8:0    0    2G  0 disk 
   sdb      8:16   0    1G  0 disk 
   └─sdb1   8:17   0 1023M  0 part 
   sdc      8:32   0 80.1G  0 disk 
   └─sdc1   8:33   0 80.1G  0 part /
   ```

3. Format the partition:

   ```bash
   sudo mkfs.ext4 /dev/sdb1
   ```

    ```output
    mke2fs 1.44.5 (15-Dec-2018)
    Creating filesystem with 262144 4k blocks and 65536 inodes
    Filesystem UUID: 3b1b7b7b-7b7b-7b7b-7b7b-7b7b7b7b7b7b
    Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (4096 blocks): done
    Writing superblocks and filesystem accounting information: done
    ```

4. Create a mount point and mount the disk:

   ```bash
   sudo mkdir /mnt/mydisk
   sudo mount /dev/sdb1 /mnt/mydisk
   ```

5. Verify the mount:

   ```bash
   df -h | grep mydisk
   ```

    ```output
        /dev/sdb1       989M   24K  922M   1% /mnt/mydisk
    ```

---

## **Part 3: Copy and Delete a Picture**

1. Copy a test image (`pxl_info.jpg`) to the mounted disk:

   ```bash
   sudo cp mypicture.jpg /mnt/mydisk/pxl_info.jpg
   cp pxl.jpg /mnt/mydisk/pxl_logo.jpg
   ```

2. Verify the file exists:

   ```bash
   ls -l /mnt/mydisk/
   ```

3. Delete the file:

   ```bash
   sudo rm /mnt/mydisk/pxl_info.jpg
   ```

4. Sync changes and unmount:

   ```bash
   sync
   sudo umount /mnt/mydisk
   ```

---

## **Part 4: Create a Forensic Disk Image**

Install dcfldd if not already installed:

```bash
sudo apt-get install dcfldd -y
```

1. Create an image of the disk using `dcfldd`:

   ```bash
   sudo dcfldd if=/dev/sdb of=forensic_image.dd bs=4096 conv=noerror,sync sizeprobe=if
   ```

    ```output
    [100% of 1024Mb] 262144 blocks (1024Mb) written. 00:00:00 remaining.
    262144+0 records in
    262144+0 records out
    ```

2. Verify the integrity:

   ```bash
   d5sum /dev/sdb > source_hash.txt
   md5sum forensic_image.dd > image_hash.txt
   cat source_hash.txt image_hash.txt
   ```

3. Compare the hashes to confirm the image is valid.

    ```bash
    md5sum -c image_hash.txt
    ```

    ```output
    forensic_image.dd: OK
    ```

---

## Mounting a Forensic Disk Image in Read-Only Mode

### **Method 1: Using mount with a loop device**

To mount a forensic disk image (`forensic_image.dd`) in **read-only** mode while specifying an offset, use the following command:

1. Create a directory for the mount:

   ```bash
   mkdir /mnt/forensic_mount
   ```

2. Mount the image:

    Check the partition table of the image:

    ```bash
    mmls -ab forensic_image.dd                                
    ```

    ```output
    DOS Partition Table
    Offset Sector: 0
    Units are in 512-byte sectors

        Slot        Start        End          Length       Size    Description
    002:  000:000   0000002048   0002097151   0002095104   1023M   Linux (0x83)
    ```

    The start sector is `2048`.

    ```bash
    sudo mount -o ro,loop,offset=$((2048*512)) forensic_image.dd /mnt/forensic_mount
    ```

    **Explanation:**
   - ro → Mounts the image as read-only to prevent modifications.
   - loop → Uses a loop device to handle the image file as a block device.
   - offset=$((2048*512)) → Specifies the starting byte offset where the partition begins:
     - 2048 is the sector where the partition starts.
     - 512 is the sector size (default for most disks).
     - This calculates the byte offset: 2048 * 512 = 1048576 (1 MiB).
   - forensic_image.dd → Path to the forensic image.
   - /mnt/forensic_mount → Target mount directory.

3. Check if the file is present:

    ```bash
    ls forensic_mount/
    ```

    ```bash
    ls /mnt/forensic_mount 
    ```

    ```output
    lost+found  pxl_logo.jpg
    ```

   - The file `pxl_logo.jpg` should be present in the mounted image.
   - The file `pxl_info.jpg` should not be present because it was deleted.

4. Unmount the image:

   ```bash
    sudo umount /mnt/forensic_mount
    ```

### **Method 2: Using losetup and mount**

1. Use `losetup` to create a loop device for the image:

    ```bash
    sudo losetup -fP forensic_image.dd 
    ```

    - -f → Find the first available loop device.
    - -P → Create partitions for the loop device.

    Find the assigned loop device:

    ```bash
    losetup -a
    ```

    ```output
    /dev/loop0: [0048]:37 (/mnt/share/vm_share/imaging/forensic_image.dd)
    ```

2. Check for Partitions

    ```bash
    lsblk
    ```

    ```output
    NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    loop0       7:0    0    1G  0 loop 
    └─loop0p1 259:0    0 1023M  0 part 
    sda         8:0    0    2G  0 disk 
    sdb         8:16   0    1G  0 disk 
    └─sdb1      8:17   0 1023M  0 part 
    sdc         8:32   0 80.1G  0 disk 
    └─sdc1      8:33   0 80.1G  0 part / 
    ```

    ```bash
    mmls -aB /dev/loop0
    ```

    ```output
    DOS Partition Table
    Offset Sector: 0
    Units are in 512-byte sectors

        Slot        Start        End          Length       Size    Description
    002:  000:000   0000002048   0002097151   0002095104   1023M   Linux (0x83)
    ```

3. Mount the partition:

    ```bash
    sudo mount -o ro /dev/loop0p1 /mnt/forensic_mount
    ```

4. Check if the file is present:

    ```bash
    ls /mnt/forensic_mount
    ```

    ```output
    lost+found  pxl_logo.jpg
    ```

    pxl_info.jpg should not be present because it was deleted.

5. Unmount the image:

    ```bash
    sudo umount /mnt/forensic_mount
    ```

    ```bash
    sudo losetup -d /dev/loop0
    ```

    ```bash
    losetup -a
    ```

    -a → Show all loop devices.

    no output should be displayed.

## **Part 5: Recover the Deleted Image**

### **Method 1: Using `foremost`**

install `foremost` if not already installed:

```bash
sudo apt-get install foremost -y
```

1. Create a directory for recovered files:

   ```bash
   mkdir recovery_output_foremost
   ```

2. Run `foremost` to extract images:

   ```bash
   foremost -t jpg -i forensic_image.img -o recovery_output_foremost/
   ```

3. Check if `pxl_info.jpg` is recovered:

   ```bash
   ls recovery_output/jpg/
   ```

### **Method 2: Using `photorec`**

Create the following directory:

```bash
mkdir recovery_output_photorec
```

1. Start `photorec`:

   ```bash
   sudo photorec forensic_image.img
   ```

2. Select:
   - **The correct partition**
   - **JPEG file type**
   - **A recovery directory**

3. Check the output folder for the recovered image.

---

### **Assessment Criteria**

| Criteria                         | Points |
|----------------------------------|--------|
| Successfully added and formatted disk | 10     |
| Copied and deleted the file        | 10     |
| Created a valid forensic image     | 20     |
| Verified integrity of the image    | 10     |
| Recovered the deleted file         | 20     |
| Report and screenshots provided    | 30     |

---

This assignment gives students **hands-on experience** with forensic imaging and recovery in Kali Linux. 🚀
