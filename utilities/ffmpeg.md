# Contents

- [streams](#streams)
- [subtitles](#subtitles)
- [conversion](#conversion)
- [presets](#presets)
- [quality](#quality)
- [output](#output)
- [expressions](#expressions)
- [usecases](#usecases)
    - [archive](#usecases#archive)
        - [music](#usecases#archive#music)
        - [podcasts](#usecases#archive#podcasts)
        - [video](#usecases#archive#video)
        - [for iOS](#usecases#archive#for iOS)
    - [batch](#usecases#batch)
    - [record audio](#usecases#record audio)
    - [record screen](#usecases#record screen)
    - [reverse](#usecases#reverse)
    - [capture USB video](#usecases#capture USB video)
    - [concat images to video](#usecases#concat images to video)
    - [cut](#usecases#cut)
    - [concat](#usecases#concat)
        - [embed cover](#usecases#concat#embed cover)
    - [convert DVD](#usecases#convert DVD)
    - [crop](#usecases#crop)
    - [exctract frames](#usecases#exctract frames)
    - [overlaying image on video](#usecases#overlaying image on video)
    - [parallel](#usecases#parallel)
    - [remove audio](#usecases#remove audio)
    - [resize](#usecases#resize)
    - [rotate](#usecases#rotate)
        - [via metadata](#usecases#rotate#via metadata)
        - [transpose](#usecases#rotate#transpose)
    - [specifying start and end frames](#usecases#specifying start and end frames)
- [filters](#filters)
    - [select](#filters#select)
    - [segmented output](#filters#segmented output)
    - [speed](#filters#speed)
        - [audio](#filters#speed#audio)
        - [video](#filters#speed#video)
        - [together](#filters#speed#together)
- [info](#info)
    - [bitrate](#info#bitrate)
    - [framerate](#info#framerate)
- [generate](#generate)
    - [color noise](#generate#color noise)
    - [monochrome noise](#generate#monochrome noise)
    - [black without sound](#generate#black without sound)
- [experiments](#experiments)

# streams
By default *ffmpeg* only selects 1 video, 1 audio and 1 subtitle track.
To preserve all tracks, explicit `map` assigment should be passed.
- `-map 0:v` preserves all video tracks
- `-map 0:a` preserves all audio tracks
- `-map 0:s` preserves all subtitle tracks
External tracks should be passed with `-i`

e.g.
* `ffmpeg -i <video> -i <audio> -map 0:v -map 0:a -c:v copy -c:a copy <output>`

# subtitles
External subtitles could be passed to `-c:s`:
* `ffmpeg ... -c:s <file> ...`
but sometimes this solution doesn't work.

More robust method is:
* `ffmpeg -i ... [-sub_charenc <enc>] -i <subtitle_file> ... -c:s <subtitle_codec> ...`
where:
- `-sub_charenc` sets text encoding (`LATIN1`, `CP1252`, `UTF-8` etc), optional
- `-c:s` sets subtitles codec:
    - for *MKV* `copy`, `ass`, `srt` `ssa`
    - for *MP4* `copy`, `mov_text`
    - for *MOV* `copy`, `mov_text`

# conversion
Codecs are determined by `-c` argument
- `-c:v copy` will copy video codec from the source
- `-c:a copy` will copy audio codec from the source
- `-c:v libx264` will convert video to *H.264*
- `-c:a libvorbis` will convert audio to *vorbis*

# presets
- `-fpre <file>` preset from file
- `-vpre <name>` named video preset, e.g. `normal`
- `-apre <name>` named audio preset
- `-spre <name>` named subtitle preset

# quality
- `-b <number><measure>` sets bitrate, e.g. `-b 4M`, `-b 64k`
- `-preset <name>` e.g. `-preset medium`
    - `veryslow` - smaller file
    - `veryfast` - faster conversion
- `-crf <number>` lower `<number>` means better quality, 15-25 is usually good

# output
- `-pix_fmt yuv420p <output>` sets pixel format of the output
- `ffmpeg -i ... -o <output> -y` will override *output* file if exists

# expressions
`select` filter (maybe others) can use expressions.

e.g. to select frame on 11 second `select=eq(t,11)`, for more info see [select](#select)
sometimes `,` in expression should be escaped with `\`
`*` is logical `AND`
`+` is logical `OR`
full list of expressions is available in [documentation](https://ffmpeg.org/ffmpeg-utils.html)

# usecases
## archive
**NOTE:** this is personal preferences for saving podcasts/videos  
- `-vbr on` option doesn't really impact filesize
- `1.8` speed is okay
- `crf 30` significantly decreases filesize
- conversion with `libvorbis` and then `libopus` produces the same file as with `libopus` from the start
- `libopus` allows to use `-ab 16k` and `-ar 16k` to decrease filesize
- `-aq 7` **increases** filesize

### music
`ffmpeg -i <input> -vn -acodec libvorbis -aq 7 <output>`

### podcasts
`ffmpeg -i <input> -vn -acodec libvorbis -filter:a "atempo=2.0" <output>`

for smaller files:
`ffmpeg -i <input> -vn -acodec libvorbis -ab 32k -ar 22050 <output>`  
or  
`ffmpeg -i <input> -vn -acodec libvopus -ab 16k -ar 16k -vbr on <output>`  

### video
`ffmpeg -i <input> -c:v libx264 -c:a libvorbis -crf 30 -preset veryslow -vf scale=-2:480 <output>`

### for iOS
`ffmpeg -i <input> -vcodec libx264 -level 3.1 -preset medium -crf 23 -x264-params ref=4 -acodec copy -movflags +faststart <output>`

## batch
`for i in *.avi; do ffmpeg -i "$i" "${i%.*}.mp4"; done`

## record audio
`ffmpeg -f pulse -i <ID of source> <out>`

`<ID of source>` - full name of sink, can be obtained via `pactl list sinks`

## record screen
video only:  
`ffmpeg -video_size 1920x1080 -framerate 25 -f x11grab -i :0.0+<x>,<y> <output>`
with audio (mic):  
`ffmpeg -video_size 1920x1080 -framerate 25 -f x11grab -i :0.0+0,0 -f alsa -ac 2 -i pulse -acodec aac -strict experimental <output>`

## reverse
`ffmpeg -i <input> -vf reverse <output`

## capture USB video
`ffplay -f video4linux2 /dev/video0`  
**NOTE**: make sure `uvcvideo` kernel module is loaded (device should be at `/dev/video0`)

## concat images to video
* `ffmpeg -r 60 -f image2 -s 1280x720 -i pic%04d.png -i MP3FILE.mp3 -vcodec libx264 -c:a copy <output>`
* `ffmpeg -framerate 10 -pattern_type glob -i '*.jpg' -c:v libx264 <output>`
where:
- `-r 60` framerate
- `-s <width>x<height>` frame size
- `-i pic%04d.png` batch of source images, filenames are padded with `4` zeros: _pic0001.png, pic0002.png, ..._

## cut
* `ffmpeg -i <source> -ss <start_time> -t <duration> -async 1 <output>`
* `ffmpeg -i <source> -ss <start_time> -to <end_time> -async 1 <output>`
`<time>` in format `HH:MM:SS`

## concat
prepare list of files::
```
file '1.mp4'
file '2.mp4'
. . .
```
run::
`ffmpeg -f concat -i list -c copy output`
hint: add `-safe 0` before `-i` if file pathes are not relative

**OR**

`ffmpeg -f concat -safe 0 -i <(for f in ./*.mp3; do echo "file '$PWD/$f'; done) <output>`

### embed cover
`ffmpeg -i input.m4a -i image.jpg -map 0 -map 1 -c copy -disposition:v:1 attached_pic output.m4a`

## convert DVD
`cat *.VOB | ffmpeg -fflags +genpts -i - -c:v copy -c:a copy -c:s copy <output.mkv>`

## crop
`ffmpeg -i <input> -filter:v "crop=w:h:x:y" <output>`

## exctract frames
* extract 3 frames `ffmpeg -i <input> -ss 00:00:07.000 -vframes 3 thumb%04d.jpg` 
* extract 1 frame per seconds `ffmpeg -i <input> -vf fps=1 thumb%04d.jpg`
* extract 1 frame every 5 seconds `ffmpeg -i <input> -vf fps=1/5 thumb%04d.jpg`
* extract keyframes `ffmpeg -i <input> -vf "select=eq(pict_type\,I)" -vsync vfr thumb%04d.jpg`
  for more info see [expressions](#expressions)

## overlaying image on video
`ffmpeg -r 60 -f image2 -s 1920x1080 -i pic%04d.png -i ~/path_to_overlay.png -filter_complex "[0:v][1:v] overlay=0:0" -vcodec libx264 -crf 25  -pix_fmt yuv420p test_overlay.mp4`

## parallel
`find $path -name ... | parallel ffmpeg -i {} -y <filters> .{}.new`
* `parallel` - GNU parallel
* output format:
  * {} - same name and path as input
  * {.} - same path and name w/o extension

## remove audio
`ffmpeg -i input -c copy -an output`

## resize
* `ffmpeg -i <input> -s 720x480 -c:a copy <output>`
* `ffmpeg -i <input> -filter:v scale=width:height -c:a copy <output>`
use `-1` instead of `height` or `width` to automatic calculation  
use `-2` if codec requires

## rotate
### via metadata
* `ffmpeg -i <input> -metadata:s:v rotate=90 -c copy <output>`
this will put a metadata tag and if a video player supports it, video will be automatically rotated on playback.

*NOTE:* if input video contains rotation tag, `ffmpeg` will automatically resize it, i.e. if input video is 640x480
and it has `rotate-90` tag, `ffmpeg` will, without any additional filters, transform it to 480x640 (without tag).
Seems like this operation doesn't consume resourses.
To prevent such behaviour, `-noautorotate` can be used before `-i`.

### transpose
* `ffmpeg -i <input> -vf "transpose=<N>" <output` where `N`:
- `0` 90CounterClockwise and Vertical Flip (default)
- `1` 90Clockwise
- `2` 90CounterClockwise
- `3` 90Clockwise and Vertical Flip
for example, for 180 degree `transpose=2,transpose=2` can be used.

*NOTE:* order of filters matters, so if `transpose` the filter is first and, for example, `overlay` filter is the second, then overlay will be applied
to transposed video and vice versa.

## specifying start and end frames
* `ffmpeg -r 60 -f image2 -s 1920x1080 -start_number 1 -i pic%04d.png -vframes 1000 -vcodec libx264 -crf 25  -pix_fmt yuv420p test.mp4`

- `-start_number` specifies what image to start at
- `-vframes 1000` specifies the number frames/images in the video


# filters
## select
[documentation](https://ffmpeg.org/ffmpeg-filters.html#select_002c-aselect)

Allows to select specified frames. Could be used with [expressions](#expressions)
* select keyframes `select='eq(pict_type\,I)'`
* select only keyframes between 10-20s timestamps `select=between(t\,10\,20)*eq(pict_type\,I)`

useful variables for expression:
* `t` time
* `n` frame number

## segmented output
[documantation](https://ffmpeg.org/ffmpeg-formats.html#segment_002c-stream_005fsegment_002c-ssegment)

Allows to split output on segments.

useful arguments:
* `segment_format format` override the inner container format, by default it is guessed by the filename extension
* `segment_list name` generate also a listfile named name
* `segment_list_entry_prefix prefix` prepend prefix to each entry, useful to generate absolute paths
* `segment_list_type type` can be `flat`, `csv`, `ffconcat`, `m3u8`. If not specified the type is guessed from the list file name suffix
* `segment_time time` set segment duration to time, the value must be a duration specification. Default value is "2"
* `segment_times times` specify a list of split points. times contains a list of comma separated duration specifications, in increasing order
* `reset_timestamps 1|0` reset timestamps at the beginning of each segment, so that each segment will start with near-zero timestamps. Useful for appropriate thumbs in mediaplayers

## speed
### audio
* `ffmpeg -i <input> -filter:a "atempo=2.0" -vn <output>`
`atempo` range is 0.5..2.0, for more than 2.0 speed:
* `ffmpeg -i <input> -filter:a "atempo=2.0,atempo=2.0" -vn <output>` 

### video
use `-an` to disable audio
* `ffmpeg -i <input> -filter:v "setpts=0.5*PTS" <output>` double speed (drop frames)
* `ffmpeg -i <input> -r 60 -filter:v "setpts=0.5*PTS" <output>` double speed (no drop frames, assuming original FR was 30 fps)
* `ffmpeg -i <input> -filter:v "setpts=2.0*PTS" <output>` slow down speed
add optical filter to add smooth
* `ffmpeg -i <input> -filter:v "minterpolate='mi_mode=mci:mc_mode=aobmc:vsbmc=1:fps=120'" <output>`

### together
`ffmpeg -i <input> -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" <output>`

# info
## bitrate
`ffprobe -v error -select_streams a:0  -show_entries stream=bit_rate -of default=noprint_wrappers=1:nokey=1 `
## framerate
`$frame_count="$(ffprobe -v 0 -select_streams v:0 -count_frames -show_entries stream=nb_read_frames -of csv=p=0 in.mp4)"`


# generate
## color noise
`ffmpeg -f rawvideo -video_size 1280x720 -pixel_format yuv420p -framerate 25 -i /dev/urandom -ar 48000 -ac 2 -f s16le -i /dev/urandom -codec:a copy -t 5 -vf hue=s=0 output`
## monochrome noise
`ffmpeg -f rawvideo -video_size 1280x720 -pixel_format yuv420p -framerate 25 -i /dev/urandom -ar 48000 -ac 2 -f s16le -i /dev/urandom -codec:a copy -t 5 output`
## black without sound
`ffmpeg -f lavfi -i color=black:s=1920x1080:r=30 -preset ultrafast -t 00:44 <output>`

# experiments
* [GPU node performance](../hardware/gpu.md#GPU-node-performance-measurment)
