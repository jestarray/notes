# notes
common CLI commands that I use
  
## Fixing grub dual boot
1: Boot arch usb and mount root partition to /mnt `arch-chroot /mnt`

2: Mount the EFI Partition to `/boot/efi`

3: Reinstall grub `grub-install` and `grub-mkconfig -o /boot/grub/grub.cfg`

## Fixing after dd or win32diskimager messes with your flash drive:

```
wipefs --all /dev/sdXX
```

## merge all mp4 files in a folder with ffmpeg without re-encoding
### windows:
Create a list of files using CMD
```
(for %i in (*.mp4) do @echo file '%i') > LIST.txt
```	
### linux:
```
for f in ./*.mp4; do echo "file '$f'" >> LIST.txt; done
```
### finally:
```
ffmpeg -f concat -safe 0 -i LIST.txt -c copy zout.mp4
```

## compressing an mp4 file with ffmpeg
### (just shove it in and the defaults mostly work(TM))
```
ffmpeg -i input.mp4 output.mp4
```

