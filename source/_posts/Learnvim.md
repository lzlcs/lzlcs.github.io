---
title: Vim 学习笔记
date: 2023-01-11 19:19:48
categories:
- Tools
tags: 
- Vim
- Tools
---

A note about learning vim.

<!--more-->

## Chapter 1: The Vim Way

### Tip 1: Meet the Dot Command

> `.` is to repeat the last change

* The Dot Command is a Micro Macro.

### Tip 2: Don't Repeat Yourself

> Move and use the dot command

* Reduce Extraneous Movement

### Tip 3: Take One Step Back, Then Three Forward  

* Make the change repeatable
* Make the motion repeatable
* All together

### Tip 4: Act, Repeat, Reverse

> Notice that many commands have its reversed form to themselves.
> So you can undo almost everything.

### Tip 5: Find and Replace by Hand

* Use `:%s/content/copy/g` to replace all in this file
  * Or use `/content` and `.` to replace one by one with your judgement.

### Tip 6: Meet the Dot Formula

* One Keystroke to Move, One Keystroke to Execute.

## Chapter 2: Normal Mode

### Tip 7: Pause with Your Brush Off the Page

> When you ask yourself whether you should enter the normal mode, then do it.

### Tip 8: Chunk Your Undos

* `u` is the undo command.
  * `u` and `<C-r>` are relative commands.

### Tip 9: Compose Repeatable Changes

* In a word, make your command repeatable as possible as you can,
  so when you find the next operator can be done by `.`, you`ll be happy.

### Tip 10: Use Counts to Do Simple Arithmetic

* `<C-a>` and `<C-x>` perform addition and substraction on numbers.

### Tip 11: Don’t Count If You Can Repeat

* You can use `dw.......` rather than `d7w` because you counting time are long.
  * Also, if you type dot one more time, you can type `u` to undo it easily.
* But you can use `d7w` to have a cleaner undo tree if you like counting.

Which to use is up to you.

### Tip 12: Combine and Conquer

* Operator + Motion = Action
* Try to map your own keys

## Chapter 3 Insert Mode

### Tip 13: Make Corrections Instantly from Insert Mode

* `<C-h>` is the same as the `<BS>`
* `<C-w>` is the same as `db`
* `<C-u>` is the same as `d^`

### Tip 14: Get Back to Normal Mode

* `<C-[>` is the same as `<esc>`
* `<C-o>` is to enter the insert normal mode.

### Tip 15: Paste from a Register Without Leaving Insert Mode

* `<C-r>{register}` is to paste text from the register.
* In my opinion, it's not better than `<C-o>p`

### Tip 16: Do Back-of-the-Envelope Calculations in Place

* `<C-r>={expression}<CR>` can calculate the value of the expression.

### Tip 17: Insert Unusual Characters by Character Code

* `<C-v>{code}` can insert some special characters.
  * To know more, see `:h i_CTRL_V_digit` for more details.
  * Also, you can use `ga` to know the code of the letter under the cursor.
  * `<C-v><Tab>` is to insert the tab rather than any spaces
    whether you use the `expandtab` option.
### Tip 18: Insert Unusual Characters by Digraph

* `<C_k>{char1}{char2}` can type digraphs.
  * Use `:digraph` for more details.

### Tip 19: Overwrite Existing Text with Replace Mode

* `R` is to enter the replace mode under the normal mode
* `r{letter}` is to replace the letter under the cursor with the {letter}.
* Overwrite Tab Characters with Virtual Replace Mode
  * If you didn't set `expandtab`, replacing a tab means that
    replace many characters with one. (Many is due to the option `tabstop`.
  * Also you can use `gR` to avoid it, use `gr` similarly.

## Chapter 4

### Tip 20: Grok Visual Mode

* Use `v` to enter the visual mode.
* Most commands are the same as themselves in the normal mode.
  * The operator commands such as `y`, it needs you to confirm operation 
    object such as `iw`, but in the visual mode, the operation object directly
    becomes the areas you have selected.

* Use `<C-g>` to enter the select mode, it is similar to the other editor.
  * When you type any printable letter, the area you have selected will be 
    deleted and you will enter the insert mode with the letter printed.

### Tip 21: Define a Visual Selection

* Use `v` to enter the character-wise visual mode.
* Use `V` to enter the line-wise visual mode.
* Use `<C-v>` to enter the block-wise visual mode.
* Use `gv` to reselect the last visual selection.

Also they can be use to change visual mode from the other visual mode.

* Use `o` to move to the other end of the selection.

### Tip 22: Repeat Line-Wise Visual Commands

* `.` command in the visual mode is to reselect the last visual selection
    and do the same things such as indenting.

### Tip 23: Prefer Operators to Visual Commands Where Possible

* You'd better use dot command in the normal mode so that it can be repeatable.

### Tip 24: Edit Tabular Data with Visual-Block Mode

* Use `<C-v>` to add the `|` in the same column for many lines.
* Use `V` to change the whole line into `-` by `r-`

### Tip 25: Change Columns of Text

* Return to the normal mode so that the change can be loaded.

### Tip 26: Append After a Ragged Visual Block

* `i` and `a` have another meanings under the visual mode.
  We'll expain it later.

## Chapter 5: Command-Line Mode

### Tip 27: Meet Vim’s Command Line

* See `:h delete`, `:h yank`, `:h put`, `:h copy`, `:h move`, 
  `:h join`, `:h normal`, `:h substitute`, `:h global` for help.

### Tip 28: Execute a Command on One or More Consecutive Lines

* See `:h range`, `:h pattern`, `:h mark` for help.

### Tip 29: Duplicate or Move Lines Using ‘:t’ and ‘:m’ Commands

* `:t` is the same as `:copy`.

### Tip 30: Run Normal Mode Commands Across a Range

* Use `:normal` to execute normal commands on the [range].

### Tip 31: Repeat the Last Ex Command

* Use `@:` to execute the last ex command.
  * `:bnext` can jump to the next buffer, but `<C-o>` can jump to the last 
    position of the cursor so that after using `@:` you can use it to reverse.
  * Also for `:bprev` and `<C-i>`.

### Tip 32: Tab-Complete Your Ex Commands

* `<C-d>` can reveal the list of possible completions.

### Tip 33: Insert the Current Word at the Command Prompt

* `\*` can find the next match for the word under the cursor.
* `<C-r><C-w>` can enter the word under the cursor in the command mode.

### Tip 34: Recall Commands from History

* `<Up>` and `<Down>` can recall history commands.
* `<C-p>` and `<C-n>` can also do that.
  * But they have a disadvantage.
  * When you type `:h ` and use the arrow keystroke, it will filter the commands
    you can try out.

* Use map to solve this problem.

```lua
map("c", "<C-p>", "<Up>", { noremap = true })
map("c", "<C-n>", "<Down>", { noremap = true })
```

* `p:` can call a window which can list the history of commands, you can use 
  `<CR>` to execute the command under the cursor.
  * You can use any command in every mode, such as gather two lines divided by
    `|` and `<CR>` to execute.
* `p/` call a window which list the search history.

* `<C-f>` in the command mode can do the same as `p:`.

### Tip 35: Run Commands in the Shell

* `:!{command}` execute commands under the terminal
* `:read !{command}` paste the outputs of commands to this buffer.
* `:write !{command}` use the content of this buffer as the input of commands.
  * Notice: `:write! {command}` is different from the previous command, 
    see `:h write!` for help.

* `:[range]!{command}` can execute commands specially for this [range].
* See `:h !` for a convenient shortcut for setting the range.

### Tip 36: Run Multiple Ex Commands as a Batch

* You can save a list of commands in a `xxx.vim`, and use `:source xxx.vim` to execute it.

* To files in `:args`, you can use `:argdo source xxx.vim` to execute every files.

## Chapter 6: Manage Multiple Files

### Tip 37: Track Open Files with the Buffer List

* When you execute `nvim [filename]`, the nvim will creat a copy of this file.
  So what you do is on this copy, you can change the real file when you save it.

* You can use wildcards to edit files, for instance, `nvim \*\.cpp` 
* Then use `:ls` to see all buffers,
  you can see a `%a` in front of your current buffer's name.
* `:bnext` and `:bprev` can change the current buffer.
* `:bfirst` and `:blast` are easy to comprehense.
* `<C-6>(<C-^>)` can change to the buffer whose name has a `#` in front of itself.

### Tip 38: Group Buffers into a Collection with the Argument List

* `:args {lists}` can add {lists} to populate argument lists.
* Use `:args` to print the argument list, with the current file in square brackets.

1. List every file's name.
2. Use wildcards.
  * `*` matches anything, including nothing
  * `**` matches anything, including nothing, recurses into directories
3. Use shell commands' outputs.
  * `:args \`cat filename.txt\``

* Use arguments list rather than buffers.

### Tip 39: Manage Hidden Files

* When a buffer is modified but not saved, `:ls` will show a `+` in front of this buffer.
* The letter before the buffers indicates status, `a` means active, `h` means hidden.

* If a hidden buffer isn't be saved, quiting vim with `:q` is not allowed.
* After seeing the words, vim will load the first unsaved file when you use `enter`.

* Use `:qa!` can quit vim without saving changes.

* `:first`, `:last`, `:next`, `:prev` can jump to other files in the arguments.

### Tip 40: Divide Your Workspace into Split Windows

* `<C-w>s`, `<C-w>v` can split a new window which has same height or width as the former window.
  * The new window will display the same buffer as previous window.
    * You can use `:edit {filename}` to edit a new file.
    * Also you can us `:split {filename}` or `:vsp {filename}` instead.


* `<C-w>w` will circle among opened windows.
* `<C-w>h/l/j/k` will jump to the h/l/j/k window.
* `<C-w>c` close the current window.
* `<C-w>o` close other windows.

* `<C-w>=` equalize width and height of all windows.
* `<C-w>_` Maximize height of the active window.
* `<C-w>|` Maximize width of the active window.
* `[N]<C-w>_` Set active window height to [N] rows.
* `[N]<C-w>|` Set active window width to [N] rows.

* You won't resize windows at most time, so you can use mouse ultimately.

* See `:h window-moving` to know more about it.

### Tip 41: Organize Your Window Layouts with Tab Pages

* Tab page can collect lots of windows, so you can open a new tab to do otherthings
  and you can come back when you want.

* `lcd {path}` can change working directory locally for the current window.
  So we can creat a new tab to edit a diffrent project such as your nvim config.

* `:windo lcd {path}` can change all windows' directory to {path}.

* `<C-w>T` can move the current window to a new tab page.
* `:tabedit {filename}` can edit {filename} in a new tab page.
* `:tabclose` and `:tabonly` are similar to `<C-w>c` and `<C-w>o`.

## Chapter 7: Open Files and Save Them to Disk

### Tip 42: Open a File by Its Filepath Using ‘:edit’

* `:pwd` can show absolute path of the current file.
* We can use relative or absolute path after `:edit`.

* `:edit %<Tab>` can print absolute path of the current file from the directory
  which ordered by `:lcd`.
* `:edit %:h<Tab>` can print absolute path of the current file's directory.

### Tip 43: Open a File by Its Filename Using ‘:find’

* `find {filename}` can search {filename} in path. (Use `<Tab>` to autocomplete)
  * Path can be set by `:set path = {path},{path},...`.(See `:h path` for help)
  * `:set path+={path}` can add {path} to the end of the former path.
    * `:set path=./**` so that every files under `.` will be included.

### Tip 44: Explore the File System with netrw

* If you use `NvimTree`, `netrw` is always been banned.

### Tip 45: Save Files to Nonexistent Directories

* `<C-g>` can show file's path and other information.
* `:!mkdir -p %:h` can creat directories so that you can save.

### Tip 46: Save a File as the Super User

* `:w !sudo tee % > {path}`

## Chapter 8: Navigate Inside Files with Motions

### Tip 47: Keep Your Fingers on the Home Row

* When you use three or more times `h`, you should consider how to optimize your operators.

### Tip 48: Distinguish Between Real Lines and Display Lines

* `j`, `k`, `l`, `h`, `$`, `^`, is used to move on real lines.
* `gj`, `gk`, `gl`, `gh`, `g$`, `g^` is used to move on display lines.

### Tip 49: Move Word-Wise

* See `:h w`, `:h e`, `:h b`, `:h ge` for help
* See `:h W`, `:h E`, `:h B`, `:h gE` for help

### Tip 50: Find by Character

* See `:h f`,`:h F`,`:h t`,`:h T`,`:h ,`,`:h ;` for help

* Always use `f/F` in the normal mode and `t/T` under the operator-pending mode.
* Always find the letter with a low frequency of occurrence, this will make you faster.

### Tip 51: Search to Navigate

* Use `/` to find patterns in this file so that you can move quickly.
* Use `d/xxx` to delete, it won't delete the first letter of {pattern}, cool.

### Tip 52: Trace Your Selection with Precision Text Objects

* See `:h text-objects` for help.
* Vim's text-objects consist of two letters.

* `ib` is the same as `i(`, `iB` is the same as `i{`.

> Text objects are the next level up. If the f{char} and /pattern <CR> commands
> are like a flying kick to the head, then text objects are like a scissors kick
> that strikes two targets with a single move.

It's funny, haha.

### Tip 53: Delete Around, or Change Inside

* Now we will discuss vim's text-objects which interact with chunks of text.
  `iw`, `iW`, `ip`, `is`. A sentense.

* As usual, `d{motion}` command tends to work well with `aw`, `as` and `ap`.
* As usual, `c{motion}` command tends to work well with `iw`, `is` and `ip`.

### Tip 54: Mark Your Place and Snap Back to It

* `m{mark}` can set a mark under the cursor.
* `'{mark}` can jump to the first non-whitespace character of the line which has mark.
* `\`{mark}` can jump to the marked position.

* `Automatic Marks` are useful, see `h: mark` for help.

### Tip 55: Jump Between Matching Parentheses

* `%` can move between opening and closing pairs of parentheses, 
  ans creat a mark call `\``, so you can move back by `\`\``.

* It is recommended to install `surround` plugin.

## Chapter 9: Navigate Between Files with Jumps

### Tip 56: Traverse the Jump List

* `:jumps` can show the jump list, these commands can be seen a jump.
  * Changing the active file for the current window.
  * Moving directly to a line number.
  * Sentense-wise and paragraph-wise motions.
  * Jumping to a mark.
  * Finding patterns.
* Use `<C-i>`, '<C-o>' can jump to the next and previous one in the jump list.

* Vim can maintain many jump list for each separate window.
* Note: `<C-i>` is the same as `<Tab>`, so if you map `<Tab>`, `<C-i>` will also be mapped.

### Tip 57: Traverse the Change List

* `:changes` can show the change list.
  * `u` and `<C-r>` can undo and redo.
  * `g;` and `g,` can move the cursor to 
    the previous and next position of changes in the change list.

* Use mark to jump:
  * `.` is the position of last change. 
  * `^` is the position of the cursor the last time of quitting insert mode.

* `gi` is the same as `'^i`.

* Vim will maintain a change list to every buffer, it is diffrent from the jump list.

### Tip 58: Jump to the Filename Under the Cursor

* `gf` can jump to the file under the cursor.
* `:set suffixesadd+=.lua` can ask vim to add the suffix to the filename when opening files.
* Combined with `:set path`, it will be useful.

### Tip 59: Snap Between Files Using Global Marks

* `m{letter}` can creat a mark so you can jump back quickly.
  * Lowercase letters creat local marks.
  * Uppercase letters creat global marks.

* Remember to mark when you want to use any command that interact with the quickfix list.

## Chapter 10: Copy and Paste.

### Tip 60: Delete, Yank, and Put with Vim’s Unnamed Register

* Transposing Characters: `xp`
* Transposing Lines: `ddp`
* Duplicating Lines: `yyp`

### Tip 61: Grok Vim’s Registers

* We can specify which register we want to use by prefixing the command with `"{register}`.
  * Commands can be delete, yank and put.

* There is a special register called black hole which will truely delete something.
  * `"_d` can do that.

* `"a` is a named register, there are 26 registers to use, they work respectively.
* `""` is a unnamed register, which commands set contents of.

* The Yank Register ("0) is only written when you use `y`, also the contents will be 
  copied to the register `""`.
* `"=` register is the expression register, when you use it, you will be orderd to 
  type expression under the command mode and it will use the result of your expression.
* "% Name of the current file
* "# Name of the alternate file
* ". Last inserted text
* ": Last Ex command
* "/ Last search pattern

### Tip 62: Replace a Visual Selection with a Register

* When you are in the visual mode, `p` is to replace the selection 
  with the contents of the specified register

* You can change two blocks of text.
  * Delete one block, select the other block and `p`, go back to `p` one more.
  * You can use mark to quickly go back.

### Tip 63: Paste from a Register

* 当你使用 `yy` 等面向行的操作时, vim 将会创建面向行的寄存器.
* 当你使用面向字符或者单词的操作时, vim 将会创建面向字符的寄存器.

* 面向字符的粘贴: 由于 `p` 和 `P` 的区别, 考虑粘贴在光标前后令人烦躁, 故在插入模式下使用 `<C-r>0`.
* 面向行的粘贴: `p` 和 `P` 会把他们粘贴到当前行之前或者之后, 同时光标落在粘贴部分的开头.
  * `gp` 和 `gP` 作用同上, 但是光标会在粘贴部分的末尾

### Tip 64: 与系统剪贴板交互

* 请使用 `"+` 寄存器来与系统剪贴板进行交互.

## Chapter 10: 宏



# 起步

* `:q` 退出, `:q!` 不保存强制退出
* `:w` 保存, `:w file.txt` 保存新建的文件并命名
* `:h` 帮助, `:h write-quit` 查看特定命令的帮助

* `nvim file.txt` 打开文件, `nvim file1.txt file2.txt file3.txt`
> nvim 在不同 `buffer` 中打开文件

* `nvim --version` 查看版本, `:version` 在 `vim` 内查看版本
* `nvim +{cmd} file.txt` 打开文件后立即执行 `{cmd}`
> 可使用 `nvim +{cmd1} +{cmd2} file.txt` 执行多个命令
> `nvim -c {cmd}` 也有相同效果

* `nvim -o2` 打开两个水平分隔窗口 
* `nvim -o5 file1.txt file2.txt` 五个水平分隔的窗口并在前两个显示 `file1.txt` 和 `file2.txt`
* `nvim -O2` 打开两个垂直分隔窗口

* `<C-z>` 用来挂起 `nvim`, 使用 `fg` 返回 `nvim`
> `:suspend` 和 `:stop` 和 `<C-z>` 有相同效果


# Buffers

* `:buffers` 查看所有 `buffer` 
> `:ls` 和 `:files` 有相同作用

* `:bn` 跳转到下一个 `buffer`, `:bp` 跳转到上一个 `buffer`
* `:b <filename>` 跳转到特定文件, `:b n` 跳转到第n个 `buffer`
* `<C-o>` 跳转到跳转列表中旧位置, `<C-i>` 跳转到跳转列表新位置, `<C-^>` 跳转到先前编辑的 `buffer`
* `:bd` 删除当前 `buffer`, `:bd n`, `:bd <filename>` 删除特定 `buffer`

* `:qa` 退出全部 `buffer`, `:wa` 保存所有 `buffer`, `:qa!` 强制退出所有 `buffer`

# Windows

* `:sp <filename>` 水平分割窗口, 并在新窗口打开文件
> `<C-w> s` 打开一个水平分割的窗口
* `:vsp <filename>` 垂直分割窗口, 并在新窗口打开文件
> `<C-w> v` 打开一个垂直分割的窗口
* `:new <filename>` 创建新窗口并打开文件

* `<C-w>j` 移动到下方窗口, `hkl` 同理
* `:buffer <buffername>` 使当前窗口显示此 `buffer`

* `:q` 关闭当前窗口, `<C-w> c` 关闭当前窗口, `<C-w> o` 关闭除当前窗口的其他窗口

# Tabs

* `:tabnew <filename>` 新 `tab`
* `:tabclose` 关闭 `tab`
* `:tabnext` 下一个, `:tabprevious` 上一个, `:tablast` 最后一个, `:tabfirst` 第一个
* `nvim -p file1.txt file2.txt file3.txt` 在多个 `tab` 中打开文件
