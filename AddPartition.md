### Partitioning
Nvidia uses its own proprietary disk structure on its devices, which look like a modification of GUID partition table(GPT) with protective MBR. Most disk partitioning softwares are unable to read Shield TV's partitions because its primary header is overwritten with non-standard GPT format.

Here are list of disk paritioning software I tried and couldn't even read the disk at all: fdisk, GParted, KDE Partition manager, diskpart and Disk Management on windows. Of course, you may format the disk and rebuild partitions, but then it won't boot from Shield TV at all.

There are two softwares can read partially:

1. GNOME Disks - the default partition software for Ubuntu, however, could read the partitions IF you boot L4T from Shield TV. It won't work if you try from another computer. Also, it is still unable to modify partitions.

2. gdisk (GPT fdisk) - gdisk can read partition 2~33 from backup header. As you can see, 33rd partition is the one that holds most of 500GB storage. (462.5 GiB out of 465.66 GiB)

![table](https://github.com/na6an/ShieldTV/blob/master/images/print%20table.png)

So, downsizing this partition to add another partition to 34th or more should be the goal here, but I get this error when I try to resize disk on gdisk directly.

![error](https://github.com/na6an/ShieldTV/blob/master/images/sector%20overlap%20error.png)


**Workaround**
One way to get around this is to rewrite the partition table using gdisk, create img file of last 10 sectors (secondary gpt) by dd command, then overwite this img on the target hdd you're trying to boot with dd command again.

Following is step-by-step procedure of what I did.  
Make sure you fully understand this hdd migration guide.  
[https://forum.xda-developers.com/shield-tv/general/guide-migrate-to-ssd-hdd-size-satv-pro-t3440195](https://forum.xda-developers.com/shield-tv/general/guide-migrate-to-ssd-hdd-size-satv-pro-t3440195)

If you don't want to go through the hassle, I have last10.bin file uploaded in this dir. With that, you can skip step 3, 4, 5, 6.

1. Copy 32nd partition files into somewhere if you want to dual boot original Shield TV android software and L4T.  
`sudo cp -a /path-to-32nd-partition/* /path-to-target`

2. Make sure to make a backup img. This isn't optional because this workaround will modify and brick the hdd temporarily. Read "Warning/Backup" part above. I had to reinstall OS on my main computer during figuring out this workaround for using dd carelessly.

I went through so many issues during flashing L4T, so I wanted to be super safe, and backed up entire 500GB on my spare 640GB hdd.

There are 2 ways to do this:
Work directly on L4T flashed Shield TV, or take out hdd physically.

 - If working on L4T, plug in external drive on the device,
dd to create image of internal hdd on external drive. For example, if Shield TV's drive path was '/dev/sda' and external drive path was '/dev/sdb', then the command would be something like `dd if=/dev/sda of=/dev/sdbb/backup.img bs=4MB status=progress`

This may take really long time if you decide to create backup entire 500GB, and especially from L4T on Shield TV.  
It takes more than 10 hours for me, so I let it run overnight. I took hdd out because I'm not that patient.

- If taking out internal hdd, make sure to be careful not damaging any cable when you do. Bottom cover of the shell can be pop out with fingernail or pry opener. I found it easier to begin from the corner closest to the power connector.  
Of course, you can use flat screw driver if you don't mind damaging the cover. You will need an hdd enclosure or sata to usb cable for this.

- Instead, you may also choose to backup fist 6899870 sectors (~3.5GB) which is 31 partitions except 32nd.  
`dd if=/path-to-disk bs=512 of=first31part.bin count=6899870 status=progress`

3. Mount the target image/disk on the host computer, open the mounting with gdisk.  
In my case, img disk mounting was sdc. `sudo gdisk /dev/sdc`

![gdisk](https://github.com/na6an/ShieldTV/blob/master/images/gdisk.png)

4. Rewrite partition table manually one by one base on the partition info above.  

 - Delete existing table: select o, create a new empty GUID partition table (GPT), then y
 - Set alignment value to 1: select x, l, 1
 - Go back to main command and add partition: select m, n, partition #, fist sector #, last sector #, and file system hex code 0700  
*gdisk doesn't read the first partition, but I added first partition info as well.
 - Repeat previous step until partition 31.
 - For 32nd and later, select file system hex code 8300 to be able to read from L4T. Make sure to select lesser number than original for last sector of 32nd partition. Original last sector of 32nd partition should be the last sector of 34 (or whatever the last partition). I want to keep 32nd partition as before, to be able to boot Shield TV content, boot L4T from 33rd and use 34th as data storage. So, this is how I allocated.

 - Just in case, I also changed every partition's name to match to the original partition: select c
 - Once all done, save to the disk: select w

5. Now create binary img file of last 10 sectors from the disk we just worked on using dd command like this.  
`dd if=/path-to-disk bs=512 skip=976773158 of=last10.bin status=progress`  
The number 976773158 is assuming it's 500GB disk. If you have different size of disk, this number should be changed, too. 

6. Now, the disk we modified using gdisk is unusable in Shield TV. Recover from backup img we created from step 2.  
`dd if=/path-to-backup/backup.img of=/path-to-target bs=4MB status=progress`

7. Finally, overwite last 10 sectors with the binary img just created in step 5.  
`dd if=/path-to-last10/last10.bin of=/path-to-target bs=512 seek=976773158 status=progress`

8. It's all done. Now, if you want Sheild TV android and did step 1, copy contents back.  
`sudo cp -a /path-to-32nd-partition-copy-files/* /path-to-resized-32nd-partition/`  
This might have to be done in L4T.

### Alternative Method
You may also manually hex edit values in GPT header without using gdisk, or perhapds even without dd by editing directly in L4T. In order to do this, you need hex editor with CRC-32 checksum capability. 

The ones I used are HxD (Windows) and Okteta (Linux). Okteta, however, can handle only up to disk size of 2GB.

### Issue Remained
Although this will allow to add more partitions, the sda33.img and sda34.img boot files in xda-developers forum didn't work. Booting runs but breaks with some error.  

I'm still looking for the solution for this issue. I will update this doc when I find working solution.  
If anyone know how to boot from 33rd or 34th partition, please let me know.
