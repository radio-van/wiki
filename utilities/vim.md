# Contents

- [configuration](#configuration)
    - [TODO](#configuration#TODO)
    - [options & variables](#configuration#options & variables)
    - [lines](#configuration#lines)
    - [tips&trics](#configuration#tips&trics)
        - [reload config](#configuration#tips&trics#reload config)
        - [backup settings](#configuration#tips&trics#backup settings)
        - [change cursor in different modes](#configuration#tips&trics#change cursor in different modes)
- [plugins](#plugins)
    - [dirs](#plugins#dirs)
    - [package managers](#plugins#package managers)
        - [vim8 built-in](#plugins#package managers#vim8 built-in)
        - [vim-plug](#plugins#package managers#vim-plug)
    - [third-party plugins](#plugins#third-party plugins)
        - [fugitive](#plugins#third-party plugins#fugitive)
            - [log changes of current file](#plugins#third-party plugins#fugitive#log changes of current file)
            - [merge conflict](#plugins#third-party plugins#fugitive#merge conflict)
        - [fzf](#plugins#third-party plugins#fzf)
- [usage](#usage)
    - [autocommands](#usage#autocommands)
    - [buffers](#usage#buffers)
        - [close](#usage#buffers#close)
        - [list](#usage#buffers#list)
        - [search](#usage#buffers#search)
        - [switch](#usage#buffers#switch)
        - [save and restore](#usage#buffers#save and restore)
        - [execute commands](#usage#buffers#execute commands)
        - [count words](#usage#buffers#count words)
    - [tabs](#usage#tabs)
        - [switching](#usage#tabs#switching)
        - [usage with buffers](#usage#tabs#usage with buffers)
        - [open files](#usage#tabs#open files)
        - [re-arrange tabs](#usage#tabs#re-arrange tabs)
    - [navigation](#usage#navigation)
        - [movements](#usage#navigation#movements)
        - [jumps](#usage#navigation#jumps)
            - [changes](#usage#navigation#jumps#changes)
            - [tags](#usage#navigation#jumps#tags)
        - [quickfix](#usage#navigation#quickfix)
    - [commands](#usage#commands)
        - [shell command](#usage#commands#shell command)
        - [silent shell command](#usage#commands#silent shell command)
        - [insert smth](#usage#commands#insert smth)
        - [abbreveation](#usage#commands#abbreveation)
    - [diff](#usage#diff)
    - [editing](#usage#editing)
        - [basic idea](#usage#editing#basic idea)
        - [increase/decrease numbers](#usage#editing#increase/decrease numbers)
        - [change search result](#usage#editing#change search result)
        - [completions](#usage#editing#completions)
        - [lines](#usage#editing#lines)
            - [wraps](#usage#editing#lines#wraps)
        - [toggle case capitalization](#usage#editing#toggle case capitalization)
        - [ex/ed](#usage#editing#ex/ed)
            - [ranges](#usage#editing#ex/ed#ranges)
            - [patterns](#usage#editing#ex/ed#patterns)
            - [commands](#usage#editing#ex/ed#commands)
            - [examples](#usage#editing#ex/ed#examples)
            - [regex](#usage#editing#ex/ed#regex)
            - [lookahead / lookbehind](#usage#editing#ex/ed#lookahead / lookbehind)
        - [tips&trics](#usage#editing#tips&trics)
            - [align part of lines](#usage#editing#tips&trics#align part of lines)
            - [reverse lines](#usage#editing#tips&trics#reverse lines)
            - [comment several lines](#usage#editing#tips&trics#comment several lines)
            - [global commands](#usage#editing#tips&trics#global commands)
            - [run normal mode commands on selected lines](#usage#editing#tips&trics#run normal mode commands on selected lines)
            - [save with sudo](#usage#editing#tips&trics#save with sudo)
            - [sort uniq lines](#usage#editing#tips&trics#sort uniq lines)
            - [time travel](#usage#editing#tips&trics#time travel)
    - [selection](#usage#selection)
    - [spellcheking](#usage#spellcheking)
    - [encryption](#usage#encryption)
    - [file](#usage#file)
        - [expand filename](#usage#file#expand filename)
        - [encoding](#usage#file#encoding)
        - [save as another file](#usage#file#save as another file)
    - [path](#usage#path)
    - [macro](#usage#macro)
        - [save recordered macro](#usage#macro#save recordered macro)
    - [registers](#usage#registers)
        - [in insert mode](#usage#registers#in insert mode)
        - [named registers](#usage#registers#named registers)
        - [special registers](#usage#registers#special registers)
    - [session](#usage#session)
    - [terminal](#usage#terminal)
        - [scroll](#usage#terminal#scroll)
        - [copy-paste](#usage#terminal#copy-paste)
    - [windows and panes](#usage#windows and panes)
        - [resize window](#usage#windows and panes#resize window)
        - [switch between horizontal and vertical splits](#usage#windows and panes#switch between horizontal and vertical splits)

# configuration
## TODO
* `wildmode=longest,list,full`
* `set undofile`
* `set suffixesadd=str`
* `:co`
* `:g <regex> norm f dw` do on each line with regex
* `:g <regex> d` del all regex
* `:v <regex> d` del all not-regex
* `inoremap ;a aaaa` paste in insert mode => breakpoint snippet
* `-"- ;1 c-oma` mark from insert mode
* `gu` ? chronologicat undo? also: `undo tree`
* `dV /` line in search?


## options & variables
* `:set` is for setting _options_
* `:let` is for setting _variables_
- to check option state use `<option>?`   
- to invert boolean options use `no<option>`

## lines
* `set number` - show line numbers (1 2 3)
* `set relativenumber` - show line numbers relative to current line (3 2 1 0 1 2 3)
* `set number relativenamber` - combine both

## tips&trics
### reload config
`:so` aka `:source` - reads any file as a sequence of vim commands

tricks:   
* `:so %` when editing `.vimrc` reloads `.vimrc`
* `:so $MYVIMRC` reloads `.vimrc` from any file

### backup settings
```vim
    " Protect changes between writes. Default values of
    " updatecount (200 keystrokes) and updatetime
    " (4 seconds) are fine
    set swapfile
    set directory^=~/.vim/swap//

    " protect against crash-during-write
    set writebackup
    " but do not persist backup after successful write
    set nobackup
    " use rename-and-write-new method whenever safe
    set backupcopy=auto
    " patch required to honor double slash at end
    if has("patch-8.1.0251")
        " consolidate the writebackups -- not a big
        " deal either way, since they usually get deleted
        set backupdir^=~/.vim/backup//
    end

    " persist the undo tree for each file
    set undofile
    set undodir^=~/.vim/undo//
```

### change cursor in different modes
```vim
  let &t_SI.="\e[5 q" "SI = INSERT mode
  let &t_SR.="\e[4 q" "SR = REPLACE mode
  let &t_EI.="\e[1 q" "EI = NORMAL mode (ELSE)

  "Cursor settings:

  "  1 -> blinking block
  "  2 -> solid block 
  "  3 -> blinking underscore
  "  4 -> solid underscore
  "  5 -> blinking vertical bar
  "  6 -> solid vertical bar
```

# plugins
## dirs
* `/plugin`
**Global** auto-loading plugins
* `/autoload`
Loading when prompted by another plugin
* `/ftdetect`
filetype detectors
* `/ftplugin`
scripts for know filetypes
* `/compiler`
determines how to run compilers and linters
* `/pack`
vim8 packages

## package managers
### vim8 built-in
`mkdir -p ~/.vim/pack/foobar/{opt,start}`   

`:helptags ~/.vim/pack/foobar/opt/<plugin>/doc`   

- `/start` - autoload
- `/opt` - load on prompt

To load _optional_ plugin `:packadd <plugin>`   

### vim-plug
Auto-install **vim-plug** manager
```vim
    if empty(glob('~/.vim/autoload/plug.vim'))
      silent !curl -fLo ~/.vim/autoload/plug.vim --create-dirs
        \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
      autocmd VimEnter * PlugInstall --sync | source $MYVIMRC
    endif
```
Make plugins section
```vim
    call plug#begin('~/.vim/plugged')
    call plug#end()
```

## third-party plugins
### fugitive

#### log changes of current file
Basically `Gclog` does two things:
- loads commits to quickfix list
- opens temp buffer with `git log` output

using that various behaviour can be obtained:
* `:Gclog -- %` loads list of commits where current file was changed into quickfix list  
  and opens `git log` output (it will contain all changes, not only for current file)
* `:Git log --patch -- %` load in temp buffer a log of changes for current file 
* `0Gclog` opens a quickfix list with all changes of the current file (w/o diffs)
  `0` actually means a whole file, particular line range can be specified.

#### merge conflict
`Gdiffsplit!`

splits order

- left (top) - **target branch** _aka master_ `2`
- middle - **working copy** with conflict markers `1`
- right (bottom) - **merge branch** _aka feature_ (the one specified in `git merge` command) `3`

```
| 2 | 1 | 3 |
```

commands

* `:diffget` `dg` - get changes from buffer
* `:diffput` `dp` - put changes to buffer
* `diffupdate` - recalculate diff colors
* `Gwrite` - overwrite the working tree with active bufferA
* `[c` - previous hunk
* `]c` - next hunk

e.g.
* `1` is active, `:diffget 2` will get changes from `2` into `1`
* `3` is active, `:diffput 1` will put changes from `3` into `1`

### fzf
* `Ctrl-T` open in new tab
* `Ctrl-X` open in new split
* `Ctrl-V` open in new vertical split

Bang `!` version of command opens `fzf` in full screen   

| command             | description                                                   |
|---------------------|---------------------------------------------------------------|
| `:Files [PATH]`     | Files                                                         |
| `:GFiles [OPTS]`    | Git files (git ls-files)                                      |
| `:GFiles?`          | Git files (git status)                                        |
| `:Buffers`          | Open buffers                                                  |
| `:Colors`           | Color schemes                                                 |
| `:Ag [PATTERN]`     | ag search result (ALT-A to select all, ALT-D to deselect all) |
| `:Rg [PATTERN]`     | rg search result (ALT-A to select all, ALT-D to deselect all) |
| `:Lines [QUERY]`    | Lines in loaded buffers                                       |
| `:BLines [QUERY]`   | Lines in the current buffer                                   |
| `:Tags [QUERY]`     | Tags in the project (ctags -R)                                |
| `:BTags [QUERY]`    | Tags in the current buffer                                    |
| `:Marks`            | Marks                                                         |
| `:Windows`          | Windows                                                       |
| `:Locate [PATTERN]` | locate command output                                         |
| `:History`          | `v:oldfiles` and open buffers                                 |
| `:History:`         | Command history                                               |
| `:History/`         | Search history                                                |
| `:Snippets`         | Snippets (UltiSnips)                                          |
| `:Commits`          | Git commits (requires `fugitive.vim`)                         |
| `:BCommits`         | Git commits for the current buffer                            |
| `:Commands`         | Commands                                                      |
| `:Maps`             | Normal mode mappings                                          |
| `:Helptags`         | Help tags                                                     |
| `:Filetypes`        | File types                                                    |

# usage
## autocommands
`au <event> <pattern> <command>`  
* `<event>` `:h autocommand-events`, e.g. `VimLeave`, `BufWritePost`
* `<pattern>` `:h autocommand-patterns`, e.g. `*.py`
* `<command>` for shell command `!<shell command>` or `silent! !<shell command>`

## buffers
Buffers are loaded files. Windows are framebuffers for buffers.

### close
`:bd buff1 buff2`, use `C-A` for expanding all names starting with smth
`<range>bd`

e.g.   
`3,5bd` would close `3, 4, 5` buffers

### list
`:buffers`, `:ls`, `:files`
`:find *name_part<tab>`

### search
`:bufdo vimgrepadd <pattern> % | copen`

### switch
* `:b <number|name>`, use `Tab` for autocomplete
* `:sb <num|name>` opens buffer in horizontal split
* `:vert sb <num|name>` opens buffer in vertical split

* `gf` goto file
* `^` returns back

* `C-^` alternate buffer (`#`)
* `<number>C-^` 
   `C-W ^` or `C-W C-^` opens buffer in new window   

* `:bfirst`, `:brewind`, `sbfirst`, `sbrewind` jumps to first buffer in list
* `:blast`, `:sblast` jumps to last buffer
* `:bnext`, `:sbnext` jumps to next buffer
* `:bprevious`, `:bNext` jumps to previous buffer

* `:bmodified`, `:sbmodified` jumps to next modified buffer

* `:ball` opens all buffers from list
* `:unhide`, `:sunhide` opens all loaded buffers

to switch to already opened buffer instead of opening new window/split set option
`switchbuf` to `useopen`

### save and restore
include `%` flag to `viminfo` option

buffers could be saved per folder with local `viminfo` file

### execute commands
`:bufdo`

### count words
`g` then `^g`

## tabs
Tabs are just another representation of the group of windows/splits

### switching
* `gt` next tab
* `gT` previous tab
* `<number>gt` tab in `<number>` position (from 1)

### usage with buffers
* `:tab ball` open each buffer in new tab
* `:tab split` copy current window to a new tab
* `C-w T` move current window to a new tab

### open files
* `:tabedit`, `:tabfind`

### re-arrange tabs
* `:tabm` moves current tab to last
* `:tabm <position>` moves current tab to position (`0` for first)

## navigation

`g` is modifier for many common commands.  

### movements
* `gj`, `gk`, `g0`, `g$` acts the same as `j, k, 0, $` but do not consider wraps.  
* `gm` moves to middle of screen within line
* `zL/zH` and `zs/ze` move screen horizontally

### jumps
* `gd` to local declaration
* `gD` to global declaration
* `g*`/`g#` search word under cursor
   (will search also for words that contain word under cursor as a part)   
* `gf` to file under cursor
* `g]` to tag definition
* `gg` to first line
* `NG` to line `N` (last line if `N` is not provided)

* `C-o` / `C-i` cycle through `:jumps`
* `g;` / `g,` cycle through `:changes`

* `C-y` / `C-e` up/down 1 line

* `gi` jumps to last insert-mode edit and switches to *insert* mode

#### changes
``.  jump to last change`

```
'[  `[                  To the first character of the previously changed
                        or yanked text.
                        
']  `]                  To the last character of the previously changed or
                        yanked text.
```

#### tags
* `Ctrl-]`, `Ctrl-click`, `g-click`, `:tag` jump to tag
* `:tselect <tag>`, `g]` list locations with a tag if multiple
* `Ctrl-t`, `:pop`, `Ctrl-right_click`, `g-right_click` jump back
* `g Ctrl-]`, `:tjump` to jump or select tag if multiple
* `:tag /<pattern>` search tag

open tags in windows, tabs, splits

* `Ctrl-W }`, `:ptag` preview tag
* `Ctrl-W ]` open tag in new window
* `Ctrl-\` open definition in new tab **?**
* `Alt-]` open definition in vertical split **?**

### quickfix
* `:copen` open the quickfix window
* `:ccl`   close it
* `:cw`    open it if there are "errors", close it otherwise (some people prefer this)
* `:cn`    go to the next error in the window
* `:cp`    go to the previous error in the window
* `:cnf`   go to the first error in the next file
* `:.cc`   go to error under cursor (if cursor is in quickfix window)

## commands
* `q:` commands history
* `@:` repeat last command
* `q/` / `q?` command-line history (commands from shell)

### shell command
`: !<shell command>`

### silent shell command
no *press Enter....* after  
`:silent !<shell command>`

### insert smth
See [registers](#in-insert-mode)

### abbreveation
`:ab <abbreveation> <expansion>`

## diff
Open desired buffers and `:windo diffthis`
To turn off diff mode `:windo diffoff`


## editing

### basic idea
`operation` `range` `object`

e.g. `ci"` = `c`hange `i`nside `"`
e.g. `dw` = `d`elete `w`ord
e.g. `ciw` = `c`hange `i`nside `w`ord
e.g. `das` = `d`elete `a`round `s`entence
e.g. `dap` = `d`elete `a`round `p`aragraph
  
### increase/decrease numbers
* `C-a` increases number under cursor (whole number) or at the right of the cursor
* `C-x` decreases number under cursor (whole number) or at the right of the cursor
* `gX-a` / `gC-x` do the same, but increase/decrease incrementaly, i.e. +1, +2, +3... **?**

### change search result
`cgn` changes the next search result

### completions
`:h ins-completion`

* `C-n` keywords from file
* `C-x C-f` list of filenames
* `C-x s` spell suggestions
* `C-x C-v` Vim keywords
* `C-x C-l` complete line

### lines
* `gJ` joins lines without spaces

#### wraps
* `gq<motion>` splits long line to fit the screen.  

### toggle case capitalization
* `~` switches case of one letter
* `g~<motion>` acts on text objects, `g~~` acts on the whole line
* `gU<motion>` toggles upper-case, `gUU` acts on the whole line
* `gu<motion>` toggles lower-case, `guu` acts on the whole line

E.g.  

* toggle case of the character under the cursor, or all visually-selected characters
`~`
* toggle case of the next three characters
`3~`
* toggle case of the next three words
`g~3w`
* toggle case of the current word (inner word – cursor anywhere in word)
`g~iw`
* toggle case of all characters to end of line
`g~$`
* toggle case of the current line (same as V~)
`g~~`

### ex/ed
*ex/ed* legacy

`:<range>/<pattern>/<command>`

#### ranges

| range | meaning                                     | example               |
|-------|---------------------------------------------|-----------------------|
| N     | line number N                               | `:21s/foo/bar/g`      |
| $     | last line                                   | `:$s/foo/bar/g`       |
| .     | current line                                | `:.w single_line.txt` |
| %     | all lines                                   | `:%s/foo/bar/g`       |
| N,M   | line range                                  | `:21,25d`             |
| .,$   | from current to the end                     | `:.,$s/foo/bar/g`     |
| .+1,$ | from line _after_ current to the end        | `:.+1,$s/foo/bar/g`   |
| .,.+5 | 6 lines: from current to +5 inclusive       | `:.,.+5s/foo/bar/g`   |
| '<,'> | last selected lines (inserts automatically) | `:'<,'>s/foo/bar/g`   |
| 'a,'b | from mark `a` to mark `b`                   | `:'a,'bs/foo/bar/g`   |

range refinement:

* `/pattern` searches for the first occur of `pattern`

* `g/pattern/` works as global (i.e. find all lines with pattern)    
e.g. `:g/foo/s/bar/zzz/g` - replace all `bar` with `zzz` in all lines containing `foo`   

* `g!/pattern/` or `:v` inverts global (i.e. find all lines without patterns)

#### patterns

| command          | description                                                                              |
|------------------|------------------------------------------------------------------------------------------|
| `/pattern/`      | next line where pattern matches                                                          |
| `?pattern?`      | previous line where pattern matches                                                      |
| `\/`             | next line where the previously used search pattern matches                               |
| `\?`             | previous line where the previously used search pattern matches                           |
| `\&`             | next line where the previously used substitute pattern matches                           |
| `0;/that`        | first line containing `that` (also matches in the first line)                            |
| `1;/that`        | first line after line 1 containing `that`                                                |
| `/foo\(bar\)\@!` | equivalent to `(?!..)` perl non-captive groups. Search for **foo** not following **bar** |

#### commands
* replace `:s/../../`
* delete `:/.../d`
* move `:/.../m`, e.g. `:/foo/m-2` moves line with `foo` 2 lines above
* join `:/.../j`
* read from file `:r`
* read from cli `:r!`
* format/filter with external command `...!`, e.g. `1,$!sort` uses extermal command `sort` (also useful `fmt`, `fold`, `intend`, e.g `{!}fmt` to format current paragraph)
* `<` / `>` make intendation
* `g&` applies previous **substitution** command to the whole document

#### examples

* `:.,+21g/foo/d` delete all lines with _foo_, from current line plus 21 more lines
* `:.,$v/bar/d` delete from here to the end lines which DO NOT contain _bar_
* `:v/\(id\|name\)/d` delete all lines not containing _id_ or _name_

combine **search range** with **substitution**

* copy the lines from the current line to the next line containing 'green' (inclusive), to the end of the buffer. 
    `.,/green/co $`
* replace all `old` in the next line in which the `apples` occurs, and the line following it. 
    `/apples/,/apples/+1s/old/new/g`
* same (.1 is .+1, and because ; was used, the cursor position is set to the line matching `apples` before interpreting the .+1). 
    `/apples/;.1s/old/new/g`
* replace all `old` in the next line in which `apples` occurs, and all lines up to and including 100 lines after the current line (where the command was entered). 
    `/apples/,.100s/old/new/g`
* replace all `old` in the first block that starts with `apples` and ends with `peaches`. 
    `/apples/` identifies the first line after the cursor containing `apples`. 
    `/peaches/` is similar (first line after the current line, not the first after `apples`). Be aware of backwards ranges. 
    The block is all lines from `apples` to `peaches`, inclusive. 
    `/apples/,/peaches/ s/old/new/g`
* Same, but `peaches` identifies the first occurrence after `apples`. 
    `/apples/;/peaches/ s/old/new/g`
* Insert `# ` at the start of each line in the first block. 
    `/apples/,/peaches/ s/^/# /g`

To do a global replace in all blocks with the same patterns, use `:g`:
    
*  Insert `# ` at the start of each line in all identified blocks. 
    `:g/apples/` identifies each line containing `apples`. 
    In each such line, `.,/peaches/ s/^/# /g` is executed 
    (the `.` means the current line, where `apples` occurs). 

    `:g/apples/,/peaches/ s/^/# /g`   

#### regex
see [regex](../developing/regex.md)

Vim's style of Regex is different from **PCRE**, all special characters must be escaped, e.g.
- **PCRE** `(foo|bar)` means `foo` or `bar`
- **Vim** `\(foo\|\bar\)` otherwise `(`, `)` and `|` will be interpreted as regular characters

#### lookahead / lookbehind
* Positive lookahead `\@=`
* Negative lookahead `\@!`
* Positive lookbehind `\@<=`
* Negative lookbehind `\@<!`

e.g.
`/foo\(bar\)\@=/` will search for `foo` following `bar` but won't include `bar` in result

### tips&trics
#### align part of lines
assuming that goal is
```
aa = 123,
bbbb = 321,
c = 222
```
=>  
```
aa    = 123,
bbbbb = 321,
c     = 222
```
with help of *GNU column*:
`% ! column -t -s= -o=`

#### reverse lines
`:g/^/m0`

#### comment several lines
- `C-v` (visual block selection)
- select lines with arrow keys
- `I` (insert to the beginning of the line)
- insert comment character (for example `#`)

#### global commands
`:g/pattern/<command>`, e.g. `:g/^A/m$` moves all lines starting `A` to the end of the file

#### run normal mode commands on selected lines
`'<,'>norm <normal mode keys>`

#### save with sudo
`:w !sudo tee %`

#### sort uniq lines
`:sort u`
untested: `:%!uniq`

#### time travel
- `earlier:<time>`, `<time>` e.g. `1m` for 1 minute. Reverts file to `<time>` back
- `later:<time>` reverts file to the state later `<time>` from now

## selection
* `o` / `O` move cursor to the beginning/end of visual selection (or corner of visual block)
* `gv` re-selects previously selected stuff

## spellcheking
activate spellcheck  
* `:setlocal spell spelllang=en_us`
keybindings:  
- `]s / [s` goto next/prev typo
- `z=` list suggestions
- `zg` mark word as good
- `zw` mark word as bad

## encryption
Encryption method can be set with `set cryptmethod=...` or `set cm=...`, `blowfish2` is recommended
* To encrypt file type `:X`, then passphrase twice
* To reset password type `:X`, then empty passphrase
**Warning** do not modify or save file that was decrypted with wrong password, otherwise it would be damaged!

## file
### expand filename
There are registers with file names:
- `%` contains current file name
- `#` contains alternate file name
`:echo @<register>` displays register contents

e.g.   
`:echo @%` gets current file, relative to working dir

   to get more specific data, `:echo expand(<expression>)` could be used
   expression is `'<register>:<modificator>'`, e.g. `:echo expand('%:p')`   
   
modificators   
- `p` expand to full path
- `h` remove last component (i.e. filename and separator)
- `t` filename and extension
- `r` removes extension
- `e` file extansion

### encoding
- `set fileencoding` - gets current encoding
- `set fileencoding=utf8` - sets encoding to be saved
- `:e! ++enc=utf8` - re-read file with specified encoding

### save as another file
`:saveas`

## path
* `cd <path>` changes current working dir to `<path>`
* `lcd <path>` does the same, but only for current window or split
* `tcd <path>` does the same, but only for current tab

switching to other window/tab in case of using `lcd/tcd` will restore their `path`  

to switch to the current file's dir `cd|lcd|tcd %:h` can be used (see [filename expand](#expand-filename))  

## macro
### save recordered macro
macro are saved into named registers, so they can be simply pasted as common string (be careful with quotes, they should be escaped)
add to config: `let @MACRO_LETTER = 'PASTE_MACRO_HERE'`

## registers
* `^r<register>` in **insert mode** or **command mode** pastes content of register, e.g. `^R%` pastes current filename
* `"<register><command>` in **normal mode** sends register to command, e.g. `"%p` pastes current filename
### in insert mode
master key is `C-r`  
then the number of register can be typed to insert it or one of the following:  
* `C-f` file under cursor
* `C-w` word under cursor
* `C-a` WORD under cursor
* `C-l` line under cursor

### named registers
- `a-z` save to register
- `A-Z` append to register

### special registers
- `"` default (`y / d / x / c / s`)
- `0` yanked text
- `1-9` stack for recent deletions
- `+` system clipboard
- `*` contains visual selection
- `%` contains current file name
- `#` contains alternate file name

## session
Vim can save current buffers, layout, etc into session file
    
    `:mksession <file>`   
    
to restore it: `:so <file>`

## terminal
### scroll
- `ctrl-W N` for scrolling
- `i` or `a` for resume as terminal
### copy-paste
`ctrl-W"` follower by **register**

## windows and panes
### resize window
* `:resize <number>`
* `:resie +<number>`
* `:resie -<number>`
* `:vertical resize <number>`

* `C-w +`, `C-w -` for horizontal split
* `C-w >`, `C-w <` for vertical split
also numbers could be used, e.g. `C-w 10+`

* `C-w _` maximum height
* `C-w |` maximum width
* `C-w =` equal sizes

### switch between horizontal and vertical splits
* `C-w` + `J` - vertical -> horizontal
* `C-w` + `H` or `L` - horizontal -> vertical
