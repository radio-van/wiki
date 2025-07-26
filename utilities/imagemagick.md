# Contents

- [create image](#create-image)
- [crop](#crop)
- [montage](#montage)
- [resize](#resize)
- [convert png to jpg](#convert-png-to-jpg)
- [fill color](#fill-color)

# create image
`convert -size WxH xc:#rrggbb <output>`

# crop
`convert -crop <width>x<height>+<x>+<y> <input> <output>`

# montage
Creates collages.
`montage *.jpg -gemetry <width>x<height>x<border>x<border> -tile <horizontal>x<vertical> <output>`

# resize
`convert <input> -resize <width>x<height><flag> <output>`
flags:
* `\!` - ignore aspect ratio
* `>` - only shrink larger images
* `<` - only enlarge smaller images
* `^` - fill area (i.e. enlarge by shortest side)
* `%` - percentage resize

# convert png to jpg
`convert <png> -background white -flatten <jpg>` fix background of transparent png

# fill color
`convert <input> -fill <color> -opaque <color> <output>`
