# iOS 6 iPhone 5C downgrade guide
This guide will show you how to install iOS 6 on your iPhone 5C, please note that it is very broken, touch does **not** work, and the device will kernel panic a few seconds after booting.
## Requirements
- **A macOS system**, you might be able to do this on Linux but I highly recommend using macOS
- [IDA Pro](https://hex-rays.com/ida-pro/) for patching the kernelcache
- **Any hex editor**, I recommend [Hex Fiend](https://hexfiend.com/)
- [Legacy iOS Kit](https://github.com/LukeZGD/Legacy-iOS-Kit.git) for the SSH ramdisk to install the iOS 6 RootFS on to the device
- [reimagine](https://github.com/danzatt/reimagine) to decrypt firmware components
- [xpwn](https://github.com/OothecaPickle/xpwn) for **xpwntool**, for decompressing and recompressing the kernelcache
