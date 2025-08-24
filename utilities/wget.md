# Contents

- [download page](#download-page)

# download page
Download whole page with external medias
```
wget 
-H, --span-hosts        # go to foreign hosts when recursive
-E, --adjust-extension  # save HTML/CSS documents with proper extensions
-K, --backup-converted  # before converting file X, back up as X.orig
-k, --convert-links     # make links in downloaded HTML or CSS point to 
-p, --page-requisites   # get all images, etc. needed to display HTML page
<url>
```
OR
`wget -mkp <url>`
whete `-m` option implies `-r`, `-N`, `-l inf`, `--no-remove-listings`
