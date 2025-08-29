# Contents

- [download page](#download-page)

# download page
Download whole page with external medias

`wget --mirror --convert-links --page-requisites --adjust-extension --no-parent <url>`
or, in short:
`wget -mkpE -np <url>`

where
- `-m --mirror` turns on recursuion, equivalent to
    - `-r` recursive with default depth to 5
    - `-l inf` changes recursion depth to infinite
    - `-N` timestamping, probably to avoid downloading same files multiple times
    - `--no-remove-listing` related to FTP
- `-k --convert-links` converts links to point to local copies
- `-p --page-requisites` download all files for proper page display
- `-E --adjust-extensions` adds `.html/.css` extension to text files without them, useful for local webserver
- `-np --no-parent` guarantees that only files below given hierarchy will be downloaded
may be also useful:
- `-H --span-hosts` go to foreign hosts when recursive
- `-K --backup-converted` before converting file X, back up as X.orig
