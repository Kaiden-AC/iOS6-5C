<link rel="stylesheet" type="text/css" href="styles.css">

# iOS 6 iPhone 5C downgrade guide
This guide will show you how to install iOS 6 on your iPhone 5C, please note that it is very broken, touch does **not** work, and the device will kernel panic a few seconds after booting.<br>

## Disclaimer
I am **not** responsible for any damage to your devices caused by following this guide. Please proceed with caution and at your own risk.<br>

## Requirements
- **A macOS system**, you might be able to do this on Linux but I highly recommend using macOS<br>
- [IDA Pro](https://hex-rays.com/ida-pro) for patching the kernelcache<br>
- **Any hex editor**, I recommend [Hex Fiend](https://hexfiend.com)<br>
- **An iPhone 5 6.x iPSW, and an iPhone 5C 7.0 iPSW**, you can get these from [The Apple Wiki](https://theapplewiki.com/wiki/Firmware)<br>
- [gnu-tar](https://formulae.brew.sh/formula/gnu-tar) to compress the RootFS<br>
- [libirecovery](https://formulae.brew.sh/formula/libirecovery) to send bootchain components<br>
- [Legacy iOS Kit](https://github.com/LukeZGD/Legacy-iOS-Kit) for the SSH ramdisk to install the iOS 6 RootFS on to the device<br>
- [reimagine](https://github.com/danzatt/reimagine) to decrypt firmware components<br>
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

Then navigate to `Other Utilities > SSH Ramdisk` and enter `11A470a` for the build number, follow the steps to boot the ramdisk, then select `Connect to SSH`<br>

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
