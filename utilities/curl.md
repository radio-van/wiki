# Contents

- [usage](#usage)
    - [auth](#auth)
    - [POST](#post)
- [cookie](#cookie)
- [tips](#tips)
    - [add line after response](#add-line-after-response)
    - [ignore SSL certificate](#ignore-ssl-certificate)
- [Usecases](#usecases)
    - [get list of IMAP folders](#get-list-of-imap-folders)
    - [post photo to Telegram](#post-photo-to-telegram)
    - [post video to Telegram](#post-video-to-telegram)
    - [post video note aka circle to Telegram](#post-video-note-aka-circle-to-telegram)

# usage
## auth
`curl -u username:password ...`

## POST
`curl --data "param1=<smth>&param2=<smth>" <url>`
`curl --data "param1=<smth>" --data "param2=<smth>" <url>`

multipart:
`curl --form "fileupload=@<text_file>" <url>`

multipart with fields and a filename:
`curl --form "fileupload=@<text_file>;filename=desired-filename.txt" --form param1=<smth> --form param2=<smth> <url>`

without data:
`curl --data '' <url>`
`curl -X POST <url>`
`curl --request POST <url>`

json:  
`curl -H "Content-Type: application/json" -d '{"param": "value"}' <url>`


# cookie
* save cookies `-c, --cookie-jar <cookie_file>`
* use stored cookies `-b, --cookie <cookie_file`


# tips

## add line after response
`-w "\n"`
## ignore SSL certificate
`-k`


# Usecases

## get list of IMAP folders
`curl --user <username>:<pass> --url "imaps://<server>/INBOX" --request "LIST \"\" \"*\""`

## post photo to Telegram
`curl -X POST "https://api.telegram.org/bot<bot_token>/sendPhoto" -F photo=@<path_to_image> -F chat_id=<chat_id>`

## post video to Telegram
`curl -X POST -F "video=@<path_to_video>" "https://api.telegram.org/bot<bot_token>/sendVideo?chat_id=<chat_id>"`  
`<chat_id>` can be obtained with `.../getUpdates` (smth must be typed in chat first)  

## post video note aka circle to Telegram
```
curl -X POST "https://api.telegram.org/bot<bot_token>" -F video_note=@<path_to_video> -F chat_id=<chat_id>
```
**NOTE** video must be 640x640
