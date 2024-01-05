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

## another way to overlay the thumbnail to the start of the video(!WARNING! it overwrites the first second with the overlayed image)
It also  re-encodes but this is the last step my video production process and in the final step, I always run it through ffmpeg to get a nice compressed version anyways
```
ffmpeg -i zcleanaudio_merged.mp4 -i 1920.png \
-filter_complex "[0:v][1:v] overlay=0:0:enable='between(t,0,1)'" \
-pix_fmt yuv420p -c:a copy \
FINAL.mp4
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

### batch extending an images canvas size(note: use convert if you don't want to overwrite the file):
```
mogrify -background transparent -gravity south -extent <WIDTH>x<HEIGHT> *.png
```

## compressing png files:
```
// WARNING! FLAT OUT REPLACES THE PNG FILES
pngquant --force --skip-if-larger --ext .png *.png
```

## blue yeti microphone output fix
Blue yeti microphones have support for sound output but I generarlly don't want to use it.
Also install `pavucontrol` as a nice gui for seeing available sound ouput for applications
```
// find the analog stereo you want to output to
pactl list sinks short

// set it as the default where n is the number
pactl set-default-sink {n}

```

## fixing windows antimalware taking all the cpu

https://discord.com/channels/239737791225790464/239737791225790464/964668873309814784

According to lhecker https://github.com/lhecker who works at microsoft:

unfortunately that's not your issue... there's a bug were the Antimalware service gets stuck in a loop trying to compact the following DB if it's corrupted:
```%ProgramData%\Microsoft\Windows Defender\Scans\mpenginedb.db*```

deleting those files is safe (actually deleting that entire "Windows Defender" directory should be safe - make backups tho)
but given how it says "0MB/s" I think it's unlikely to be the culprit...
in either case: if your laptop is part of a corporate network (for instance from a school) it could be your IT department being inexperienced
there are certain settings you can make that cause extremely expensive and intrusive AV scanning (and they're better left disabled)

## Run files over a cli program with racket:
```scheme
#lang racket

(define curr-dir (current-directory))
(define files
  (filter
   (lambda (path)
     (define ext (path-get-extension path))
     ;(println ext)
     (cond
       [(boolean? ext) #f]
       [else
        (bytes=? ext #".json")]))
   (directory-list curr-dir)))

(println files)

(define (filename-no-ext p)
  (define ext (path-get-extension p))
  (define str (cond [(string? p) p]
                    [else
                     (path->string p)]))
  (cond
    [(boolean? ext) (path->string p)]
    [else
     (substring str 0 (- (string-length str) (bytes-length ext)))]))
#;
(for
    ([p files])
  (define file (path->string p))
  (define full-path (build-path curr-dir p))
  ; call the cli program you want to run it through
  ; use whereis on linux to find it
  (println p)
  (system* "/usr/bin/pdftoppm" "-png" (path->string full-path) (filename-no-ext file)))

```

## configure static ip on arch during installation:
https://bbs.archlinux.org/viewtopic.php?id=265008

1 - manually stopped  systemd-networkd.service and systemd-resolved.service services

```
systemctl stop systemd-networkd.service systemd-resolved.service
```

2 - removed any auto-associated ip addr / ip routes.
```
ip address del 192.168.0.55/24 dev enp0s2
ip route del 192.168.0.0/24 via 192.168.43.223 dev enp0s2
```

3 - configured static IP address with dhcpcd:
```
dhcpcd -S ip_address=192.168.1.23/24 -S routers=192.168.1.1 -S domain_name_servers=192.168.1.1 -s 192.168.1.23/24 enp0s2
```

## fix archlinux keyring issues
when simply updating the keyring doesn't work
```
killall gpg-agent
rm -rf /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman -S archlinux-keyring
```
