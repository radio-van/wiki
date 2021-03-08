# Contents

- [crop](#crop)
- [montage](#montage)

# crop
`convert -crop <width>x<height>+<x>+<y> <input> <output>`

# montage
Creates collages.
`montage *.jpg -gemetry <width>x<height>x<border>x<border> -tile <horizontal>x<vertical> <output>`

# resize
`convert <input> -resize <width>x<height>^ <output>`
* `^` - fit the shortest dimension of `<input>` to given box
