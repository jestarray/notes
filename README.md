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

## merging new audio over an existing mp4 file(audacity noise reduction patch in)
```
ffmpeg -i IN.mp4 -i IN.ogg -c copy -map 0:v:0 -map 1:a:0 OUT.mp4
```

## cutting out the first pre-padded 10 seconds for noise reduction:
```
ffmpeg -ss 00:0:10 -i IN.mp4 -c copy OUT.mp4
```

## cutting a video passing in duration params:
(cut from 00:00 to 02:39)
```
ffmpeg -ss 00:00:00 -i in.mp4 -to 00:02:39 -c copy -copyts out.mp4
```

## compressing an mp4 file with ffmpeg
### (just shove it in and the defaults mostly work(TM))
```
ffmpeg -i input.mp4 output.mp4
```

