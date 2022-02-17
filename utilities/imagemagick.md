# Contents

- [crop](#crop)
- [montage](#montage)
- [resize](#resize)
- [convert png to jpg](#convert png to jpg)
- [fill color](#fill color)

# crop
`convert -crop <width>x<height>+<x>+<y> <input> <output>`

# montage
Creates collages.
`montage *.jpg -gemetry <width>x<height>x<border>x<border> -tile <horizontal>x<vertical> <output>`

# resize
`convert <input> -resize <width>x<height>^ <output>`
* `^` - fit the shortest dimension of `<input>` to given box

# convert png to jpg
`convert <png> -background white -flatten <jpg>` fix background of transparent png

# fill color
`convert <input> -fill <color> -opaque <color> <output>`
