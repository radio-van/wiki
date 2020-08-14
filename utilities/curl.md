## Contents =
    - [[#usage|usage]]
        - [[#usage#POST|POST]]
        - [[#usage#POST with json data|POST with json data]]
    - [[#tips|tips]]
        - [[#tips#add line after response|add line after response]]

## usage =

### POST ==
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

### POST with json data ==

required options: ::
* `--header "Content-Type: application/json"`
* `--header "X-CSRFtoken: <token>"`
* `--referer "<base_url>"`
* `--cookie "<cookie>"`
* `--data "{<json>}"`

## tips =

### add line after response ==
`-w "\n"`
