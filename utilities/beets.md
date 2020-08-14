# Contents

- [plugins](#plugins)
    - [convert](#convert)
    - [embedart](#embedart)
- [tips & tricks](#tips-tricks)
    - [fix yaml.load bug](#fix-yamlload-bug)
    - [batch modifying attributes](#batch-modifying-attributes)

# plugins
## convert
notes:: _requires ffmpeg_
configuration::
* `auto` convert on import. default: `no`
* `tmpdir` temp storage during conversion
* `copy_album_art` copy covers when transcoding albums. default: `no`
* `album_art_maxwidth` downscale covers
* `dest` destination directory
* `embed` embed album arts. default: `yes`
* `max_bitrate` reduce bitrate to given number. files w/ lower bitrate will be copied
* `no_convert` does not convert items matching `QUERY`
* `never_convert_lossy_files` no cross-conversion. default: `no`
* `paths` dir structure in destination
* `quiet` do not print filenames. default: `false`
* `threads` parallel encoding. default: auto-detect processors number
* `format` output format. default: `mp3`
* `formats` dict with available formats
```
   convert:
     format: aac
     formats:
       aac:
         command: ffmpeg -i $source -y -acodec aac $dest
         extension: m4a
```
:: all formats can be checked with `--formats`
usage:: `beet convert [-f format] QUERY`
options::
- `-d` destination (`dest` in config)
- `-y` w/o confirmation
- `-a` match albums instead of tracks
- `-k` `--keep-new` replace files with converted ones, move old ones to destination folder
- `--pretend` test before conversion

## embedart
notes:: _requires ImageMagic_
configuration::
:: `auto` embed on import
:: `compare_threshold` similarity with existing art
:: `ifempty` avoid embedding if art already embedded
:: `maxwidth` max width for downscaling (image files are not altered)
:: `remove_art_file` remove image file
clear:: `beet clearart QUERY`
extract:: `beet extractart -o FILE QUERY` (`-y` w/o confirmation)
:: `beet extractart [-a] [-n FILE] QUERY`
:: `FILE` art filename (automatic extension)
:: `-a` associate with albums

# tips & tricks
## fix yaml.load bug
replace::
```
   yaml.load(...)
```
:: with
```
   yaml.safe_load(...)
   yaml.full_load(...)
   yaml.unsafe_load(...)
```

## batch modifying attributes
```
find ./ -type f -name '*.m4a' -exec bash -c 'ffprobe -v error -select_streams a:0  -show_entries stream=bit_rate -of default=noprint_wrappers=1:nokey=1 "$1" | xargs -I % beet modify -y bitrate='%' "$1"; echo "$1"' _ {} \;
```
