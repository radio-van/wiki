# Contents

- [usage](#usage)
    - [POST](#post)
        - [post video to Telegram](#post-video-to-telegram)
    - [POST with json data](#post-with-json-data)
- [tips](#tips)
    - [add line after response](#add-line-after-response)

# usage
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

### post video to Telegram
`curl -X POST -F "video=@<path_to_video>" "https://api.telegram.org/bot<bot_token>/sendVideo?chat_id=<chat_id>"`  
`<chat_id>` can be obtained with `.../getUpdates` (smth must be typed in chat first)  

## POST with json data

required options: ::
* `--header "Content-Type: application/json"`
* `--header "X-CSRFtoken: <token>"`
* `--referer "<base_url>"`
* `--cookie "<cookie>"`
* `--data "{<json>}"`

# tips

## add line after response
`-w "\n"`
