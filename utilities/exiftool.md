# Contents

- [tags](#tags)
    - [exclude metadata](#exclude-metadata)
    - [show only value](#show-only-value)
- [tips & tricks](#tips-tricks)
    - [remove all metadata except date and location](#remove-all-metadata-except-date-and-location)

# tags
## exclude metadata
--TAG::
:: Exclude specified tag from extracted information. Same as the `-x` option.
    May also be used following a `-tagsFromFile` option to exclude tags from being copied,
    or to exclude groups from being deleted when deleting all information,
    i.e. `"-all= --exif:all"` deletes all but EXIF information.
    But note that this will not exclude individual tags from a group delete.
    Instead, individual tags may be recovered using the `-tagsFromFile` option,
    i.e. `"-all= -tagsfromfile @ -artist"`.
    Wildcards are permitted as described above for `-TAG`. 
```
  exiftool -overwrite_original -all= -tagsFromFile @ -<tag_to_preserve> *.jpg
```

    What the command does?<br>
    `-all=` deletes all the tags<br>
    `-tagsFromFile @` takes the listed flags from the source file,<br>
    in this case `@` represents the current file and writes them to the destination.

## show only value
`exiftool -s3 -TAG <file>`


# tips & tricks

## remove all metadata except date and location
```
  exiftool -all= -tagsFromFile @ -GPS* -File* -*Date* <file>.jpg
```
