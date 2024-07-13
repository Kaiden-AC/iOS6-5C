<link rel="stylesheet" type="text/css" href="styles.css">

# iOS 6 iPhone 5C downgrade guide
This guide will show you how to install iOS 6 on your iPhone 5C, please note that it is very broken, touch does **not** work, and the device will kernel panic a few seconds after booting.<br>

## Disclaimer
I am **not** responsible for any damage to your devices caused by following this guide. Please proceed with caution and at your own risk.<br>

## Note
When I put stuff in `<>` it means an action, so `<enter>` means you would press enter, `<default value - 4>` means you type the default value but subtract 4.<br>

## Requirements
- **A macOS system**, you might be able to do this on Linux but I highly recommend using macOS<br>
- [IDA Pro](https://hex-rays.com/ida-pro) for patching the kernelcache<br>
- **Any hex editor**, I recommend [Hex Fiend](https://hexfiend.com)<br>
- **An iPhone 5 6.x iPSW, and an iPhone 5C 7.0 iPSW**, you can get these from [The Apple Wiki](https://theapplewiki.com/wiki/Firmware)<br>
- [gnu-tar](https://formulae.brew.sh/formula/gnu-tar) to compress the RootFS<br>
- [libirecovery](https://formulae.brew.sh/formula/libirecovery) to send bootchain components<br>
- [Legacy iOS Kit](https://github.com/LukeZGD/Legacy-iOS-Kit) for the SSH ramdisk to install the iOS 6 RootFS on to the device<br>
- [reimagine](https://github.com/danzatt/reimagine) to decrypt firmware components<br>
- [image3maker](https://github.com/darwin-on-arm/image3maker) to repack img3 images
- [iBoot32Patcher](https://github.com/iH8sn0w/iBoot32Patcher) to patch iBoot components<br>
- [xpwn](https://github.com/OothecaPickle/xpwn) for **xpwntool** and **dmg**, we will **xpwntool** to decompress and recompress the kernelcache, and **dmg** to decrypt the RootFS<br>

## Preparations
First decrypt the RootFS DMG, you can get firmware keys and file names from [The Apple Wiki](https://theapplewiki.com/wiki/Firmware)<br>
`dmg extract encrypted.dmg extract.dmg -k <key>`<br>

Then convert it to UDZO format<br>
`dmg build extract.dmg udzo.dmg`<br>

Mount the DMG, take note of the mount point<br>
`hdiutil attach udzo.dmg`<br>

Enable ownership on the volume<br>
`sudo hdiutil enableOwnership <mountpoint>`<br>

Create a tar from the volume<br>
`sudo gtar -cvf fw.tar -C <mountpoint> .`<br>

## Partitioning
First, we need to boot the SSH ramdisk, enter DFU mode on your device and run Legacy iOS Kit<br>
`./restore.sh`<br>

Then navigate to **Other Utilities > SSH Ramdisk** and enter **11A470a** for the build number, follow the steps to boot the ramdisk, then select `Connect to SSH`<br>

Now once we are in the ramdisk, we need to partition the disk<br>
`gptfdisk /dev/rdisk0s1`<br>

Delete the existing partitions<br>
`d <enter> 1 <enter> d <enter>`<br>

Now create the new partitions<br>
`n <enter> 1 <enter> <enter> 524294 <enter> <enter>`<br>
`n <enter> <enter> <default value - 4> <enter> <enter>`<br>

Rename the new partitions<br>
`c <enter> 1 <enter> System <enter>`<br>
`c <enter> 2 <enter> Data <enter>`<br>

Write the new partition table<br>
`w <enter> Y <enter>`<br>

Now we need to create filesystems<br>
`/sbin/newfs_hfs -s -v System -J -b 4096 -n a=4096,c=4096,e=4096 /dev/disk0s1s1`<br>
`/sbin/newfs_hfs -s -v Data -J -P -b 4096 -n a=4096,c=4096,e=4096 /dev/disk0s1s2`<br>

## Extracting RootFS
Mount the System partition<br>
`mount_hfs /dev/disk0s1s1 /mnt1`<br>

Extract the RootFS tar over SSH<br>
`cat fw.tar | ssh -p 6414 root@localhost "cd /mnt1; tar xvf -"`<br>
*Note: If you get an error, you may have to add -oHostKeyAlgorithms=+ssh-dss after the port number*<br>

After that completes, we need to move files to the Data partition, first mount the Data partition<br>
`mount_hfs /dev/disk0s1s2 /mnt2`<br>

Now move files from /private/var<br>
`mv -v /mnt1/private/var/* /mnt2`<br>

Unmount both partitions and reboot the device<br>
`umount /mnt1 /mnt2`<br>
`reboot_bak`<br>

## Patching boot components
First decrypt iBSS and iBEC from your iPhone 5C 7.0 iPSW<br>
`reimagine iBSS.boardconfig.RELEASE.dfu iBSS.raw -iv <iv> -k <key> -r`<br>
`reimagine iBEC.boardconfig.RELEASE.dfu iBEC.raw -iv <iv> -k <key> -r`<br>

Patch the iBSS and iBEC<br>
`iBoot32Patcher iBSS.raw iBSS.patched`<br>
`iBoot32Patcher iBEC.raw iBEC.patched -b "-v amfi=0xff cs_enforcement_disable=1"`<br>

Pack the iBSS and iBEC into an img3 container<br>
`image3maker -f iBSS.patched -t ibss -o iBSS.img3`<br>
`image3maker -f iBEC.patched -t ibec -o iBEC.img3`<br>

Decrypt the DeviceTree from your iPhone 5 6.x iPSW<br>
`reimagine DeviceTree.boardconfig.img3 devicetree.img3 -iv <iv> -k <key>`<br>

Decrypt the kernelcache from your iPhone 5 6.x iPSW<br>
`reimagine kernelcache.release.boardconfig kernelcache.dec -iv <iv> -k <key>`<br>

Decompress the kernelcache from your iPhone 5 6.x iPSW<br>
`xpwntool kernelcache.release.boardconfig kernelcache.raw -iv <iv> -k <key>`<br>

Open your decompressed kernelcache in IDA Pro, make sure your settings are the same as below when opening it<br>

![IDA Pro settings for kernelcache](images/kernelcache-settings-ida.png)<br>

Once the file is open, navigate to `Edit > Select all` in the toolbar, then press **C**, then click **Analyze**.<br>
*Note: If it asks "Undefine already existing code/data?" click Yes*<br>

Once the kernelcache is fully analyzed, navigate to **Search > Text...**, make sure "Find all occurrences" is checked, and search for `could not find system ID`.<br>
After the search is finished, you should have 4 results<br>

![IDA Pro search results](images/search-results-ida.png)<br>

Double click the first result, you should see something like this<br>

![IDA Pro could not find system ID function](images/could-not-find-system-id-function-ida.png)<br>

Place your cursor just before `BL` and switch to hex view<br>

![IDA Pro BL hex](images/bl-hex-ida.png)<br>

First take note of just the 4 bytes that are highlighted, then copy all the bytes on that line, open the decompressed kernelcache in your hex editor and find where those bytes are, for Hex Fiend press **Option + F**, make sure in the upper left corner it is set to **Hex**, paste the bytes in and press **Next**<br>

<p align="left">
  <img src="images/bl-hex-hex-fiend.png" alt="Hex Fiend BL hex" style="background-color: transparent;" />
</p>

Replace the 4 bytes that were highlighted in IDA Pro with 00BF00BF in your hex editor

Do **exactly the same** process, but this time searching for `XIP is still set` in IDA Pro


