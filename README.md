# notes
common CLI commands that I use

## Gimp distorting layer colors
Image -> Mode -> RGB
  
## Fixing grub dual boot when windows messes it up
1: Mount root paartition to `/mnt`, e.g `mount /dev/sda3 /mnt`

2: Chroot into it: `arch-chroot /mnt`

3: Mount EFI partition: `mount /dev/sda1 /boot/EFI`

4: Reinstall grub `grub-install` and `grub-mkconfig -o /boot/grub/grub.cfg`

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
### (just shove it in and the defaults mostly work(TM)), second option for further compression ala martins
```
ffmpeg -i input.mp4 output.mp4

ffmpeg -i IN.mp4 -c:v libx264 -profile:v high -preset:v slow -crf:v 24 -c:a aac -b:a 128k -movflags +faststart OUT.mp4
```

## creating a video file from an image file using ffmpeg
https://stackoverflow.com/questions/24102336/how-can-i-place-a-still-image-before-the-first-frame-of-a-video
```
//-t controls the seconds
ffmpeg -loop 1 -framerate 30 -i image.png -c:v libx264 -t 1 -pix_fmt yuv420p image.mp4
```
## to splice in a still image.mp4 file into the front of the video:
```
//convert the image.mp4 to an image.ts file
ffmpeg -i image.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts image.ts

//and the said video also
ffmpeg -i video.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts video.ts

//finally stick it infront of the video
ffmpeg -i "concat:image.ts|video.ts" -c copy -bsf:a aac_adtstoasc output.mp4
```


## image editing with imagemagick

### mirror in the vertical axis:
warning! edits the file in place and overwrites
```
mogrify -flip *.jpg
```
### mirror in the horizontal axis:
```
mogrify -flop *.jpg
```
