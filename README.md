# quick-linux
Awesome instructions for installing Linux distributions, quickly.

## How to choose instructions
So, you know your distribution, but not sure which of the instructions you should be using? Follow these instructions to ascertain it!

### Already have an OS on this PC
#### Windows
Open a `CMD`, and run:

```
echo %PROCESSOR_ARCHITECTURE%
```

If it says `AMD64` or `x86_64`, choose any instruction that has the `amd64` tag (example, `amd64-*-*.md`).

Next, open "System Information", and under BIOS Mode, you can find the boot mode. If it says Legacy, you want anything that's marked with `*-*-bios`. If it says UEFI, you want anything marked with `*-*-efi`.

#### Linux
```
uname -m
```

If it says `AMD64` or `x86_64`, choose any instruction that has the `amd64` tag (example, `amd64-*-*.md`).

Now, run 
```
ls /sys/firmware/efi
```

If it returns an error, you're using BIOS - so you want anything marked with `*-*-bios`. If it's an actual folder, you want anything with `*-*-efi`.
