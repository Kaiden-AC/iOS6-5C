<link rel="stylesheet" type="text/css" href="styles.css">

# iOS 6 iPhone 5C downgrade guide
This guide will show you how to install iOS 6 on your iPhone 5C, please note that it is very broken, touch does **not** work, and the device will kernel panic a few seconds after booting.
## Disclaimer
I am **not** responsible for any damage to your devices caused by following this guide. Please proceed with caution and at your own risk.
## Requirements
- **A macOS system**, you might be able to do this on Linux but I highly recommend using macOS
- [IDA Pro](https://hex-rays.com/ida-pro) for patching the kernelcache
- **Any hex editor**, I recommend [Hex Fiend](https://hexfiend.com)
- **An iPhone 5 6.x iPSW, and an iPhone 5C 7.0 iPSW**, you can get these from [The Apple Wiki](https://theapplewiki.com/wiki/Firmware)
- [gnu-tar](https://formulae.brew.sh/formula/gnu-tar) to compress the RootFS
- [libirecovery](https://formulae.brew.sh/formula/libirecovery) to send bootchain components
- [Legacy iOS Kit](https://github.com/LukeZGD/Legacy-iOS-Kit) for the SSH ramdisk to install the iOS 6 RootFS on to the device
- [reimagine](https://github.com/danzatt/reimagine) to decrypt firmware components
- [iBoot32Patcher](https://github.com/iH8sn0w/iBoot32Patcher) to patch iBoot components
- [xpwn](https://github.com/OothecaPickle/xpwn) for **xpwntool** and **dmg**, we will **xpwntool** to decompress and recompress the kernelcache, and **dmg** to decrypt the RootFS
## Preparations
First decrypt the RootFS DMG, you can get firmware keys and file names from [The Apple Wiki](https://theapplewiki.com/wiki/Firmware)
`dmg extract encrypted.dmg extract.dmg -k <key>`
Then convert it to UDZO format
`dmg build extract.dmg udzo.dmg`
Mount the DMG, take note of the mount point
`hdiutil attach udzo.dmg`
Enable ownership on the volume
`sudo hdiutil enableOwnership <mountpoint>`
Create a tar from the volume
`sudo gtar -cvf fw.tar -C <mountpoint> .`
