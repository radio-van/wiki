# Contents

- [usage](#usage)
    - [auth](#usage#auth)
    - [POST](#usage#POST)
- [cookie](#cookie)
- [tips](#tips)
    - [add line after response](#tips#add line after response)
    - [post video to Telegram](#tips#post video to Telegram)

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
`curl -H "Content-Type: application/json" -d "{<json>}" <url>`

# cookie
* save cookies `-c, --cookie-jar <cookie_file>`
* use stored cookies `-b, --cookie <cookie_file`

# tips

## add line after response
`-w "\n"`

## post video to Telegram
`curl -X POST -F "video=@<path_to_video>" "https://api.telegram.org/bot<bot_token>/sendVideo?chat_id=<chat_id>"`  
`<chat_id>` can be obtained with `.../getUpdates` (smth must be typed in chat first)  
