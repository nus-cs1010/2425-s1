# Vim HOWTO: Common Code Editing Operations

## Vim as a Source Code Editor

A source code editor is more than a text editor.  A source code editor is designed and optimized for common operations performed by a programmer while programming.  In this article, we summarize some common operations that you will likely perform and how you will perform them in Vim.

## Code Browsing

!!! Quote "Reading vs. Writing"

    “.. the ratio of time spent reading versus writing is well over 10 to 1. We are constantly reading old code as part of the effort to write new code." 

    ― Robert C. Martin, Clean Code: A Handbook of Agile Software Craftsmanship

As a programmer, we spend a significant amount of time reading source code.  This behavior explains why Vim starts in normal mode for reading and optimizes many shortcuts for navigating source code.  

### Navigation Within a File

_You need to go beyond using arrow keys to move character-by-character or line-by-line for better efficiency and productivity._  Some useful operations are:

| Operation       | Cursor Position | Forward        |  Backward   | |
|-----------------|-|----------------|-------------|-|
| Move word-by-word | Anywhere | w | b | |
| Move page-by-page | Anywhere | CTRL-f | CTRL-b | |
| Move block-by-block | Anywhere | { | } |  A block (or paragraph) is defined as consucutive non-empty lines.  This is useful for instance to navigate method-by-method, provided you have blank lines between two methods |
| Next occurance of word | A word | * | # | Useful for looking for where a variable or method is defined/used |
| Matching bracket | (, ), [, ], {, }, <, or > | % | % | Jump to the matching opening/closing bracket.  Useful for quickly jumping to the start/end of a code block or to check for balanced brackets. |
| Beginning/End of the file | Anywhere | G | gg | Useful for jump to the end of the file (e.g., to append a new method) or to the beginning of the file (e.g., to add `import` statements)
| Beginning/End of the line | Anywhere | 0 | $ | Useful for jumping to the end of the line to insert comment or to the beginning to comment out the whole line.

!!! Tips "&lt;count&gt;-&lt;movement&gt;" 
    Instead of repeating the commands above $k$ times, you can issue the command $k$ followed by the movement.  For instance, 6w would move forward by six words.  4-CTRL-f would move forward by 4 pages.

To jump to a particular line, issue the command `:linenumber`.  For example, `:42` would jump to Line 42.

## Looking for Things

When reading code, we often need to look for a specific variable, method, or type.  For instance, you might wonder, what a method does, or which line sets a field to `null`.  In such scenarios, it is useful to ask Vim to search for it, rather than just scrolling through the code and eyeballing it yourself.

### Searching for a String in the Current File

To search for a string, type, in NORMAL mode, `/` followed by the string you are looking for.  The cursor will jump to the first occurrence of the string after the current cursor.

You can use `n` or `N` to jump to the next or previous occurrence of the string respectively.

Another useful key is `#` or `*`, already mentioned above, for searching the current word the cursor is under.

## Comparing Two Files

You can compare two files with Vim, using the `-d` flag. For instance,

```
vim -d file1 file2
```

would open up two files for line-by-line comparison. This is most useful if you want to compare the output of your program with the expected output and figure out the difference.

## Moving Things Around

Often we need to move statements/blocks/methods/classes around, either to fix bugs or to tidy up our code.  We can do that in Vim NORMAL or VISUAL mode.  This often involves two steps: (i) cutting/deleting what you want to move at the source, and (ii) pasting it to the destination.

### Swapping lines

Suppose we have two statements that are out of order.  For instance, we might write
```C
i = 2 * j;
int j = 0;
```
which does not compile.  We should, of course, declare and initialize `j` first before using it.  Placing the cursor on the line `i = 2 * j`, we can perform the two steps above. (i) Type `dd` to delete the line (essentially move it into the clipboard).  The cursor will move to the line `int j = 0`.  (ii) Type `p` to paste it after the current line.

`ddp` essentially swaps two lines.

You can prefix the command with a number $k$ to delete $k$ lines.

### Reordering Multiple Lines

If you want to cut more lines than you can/willing to count, VISUAL mode is a great way to select the lines to cut.  Place the cursor at the beginning of the line you want to cut.  Press SHIFT-V to enter VISUAL-LINE mode.  Now, use any of the movement commands to move and select the lines you want to cut (remember, if possible you should avoid using arrow keys or `j` or `k` to move line-by-line).   Press `d` to delete.

Now navigate to where you want to paste and press `p`.

Note that you can navigate to a different file to paste.  This is useful if you want to move a method from one class to another.

!!! Tip "Reformatting After Pasting"

    After pasting a block of code, the indentation of the pasted code might be inconsistent.  Make it a habit to use `gg=G` to reindent your code after pasting.

### Copy-Pasting

If you need to copy-paste your code, you should first pause and think about whether you can reduce repetition in your code by factoring out the common part as a function.

If there is a valid reason to copy-paste, then you can use `y` (stands for yank) for copying text to the clipboard for pasting elsewhere.  `y` behaves similarly to `d`:

- `yy` would copy the current line
- `y` in VISUAL mode would copy the selected lines.

## Commenting Multiple Lines Of Code

You can edit multiple lines in VISUAL BLOCK mode.  This is useful for commenting and uncommenting multiple lines of code.

First, move the cursor at the beginning of a line.  Go into VISUAL BLOCK mode with CTRL-V or CTRL-Q[^1], then select the lines that you want to comment.

Type SHIFT-I to insert in VISUAL BLOCK mode, type `// ` and then `ESC` to go back to NORMAL mode.  The text `// ` would be inserted in front of each line selected.

To uncomment, select `// ` on each line that you wish to uncomment, and `x` to delete them.

## Changing Names

### Changing the name of one type/variable/method call

Occasionally, we mix up our variable or our method name, and we need to fix it before the code runs correctly or compiles.  Suppose we have

```C
double perimeter = get_area();
```

and we realize that we should be calling `get_perimeter` instead.  Instead of using `backspace` or `delete` to delete the characters one by one, we can use `cw` to change the word `get_area` into `get_perimeter`.

To do so, (i) place the cursor at the beginning of `get_area`.  Remember to avoid using arrow keys or `h` or `l` to move letter-by-letter.  You can use `w` or `b` for faster word-by-word navigation.   (ii) type `cw` to remove the word `get_area` and enter INSERT mode.  Now type `get_perimeter` to replace the method name and ESC to return back to NORMAL mode.

### Changing multiple names on the same line

Sometimes we have multiple occurrences that we wish to change on the same line.  Let's say:

```C
int area = get_area();
```

and we realize that we should be calculating `perimeter` instead of `area`.

One option is to use `cw` twice.  But we could also use the substitute command, like so.

Place the cursor on this line, and type `:s/area/perimeter/gc`, and then ENTER.  Here is what it does:

- `:` allows us to issue a command to Vim
- `s/<what to substitute>/<substitute with this>/` tells Vim what we want to replace and replace with what.
- `g` stands for `global` and it says that we want to substitute all occurrences 
- `c` is optional, and it tells Vim to confirm every replacement with us.

### Changing multiple occurrences in a block

Let's say that we have the following code:

```C
  void update_time(int *time, int now) {
      if (*time > 1200) {
          *time = now;
      } else {
          *time = 1200;
      }
  }
```

and we wish to rename the name of the parameter `time` to `start_time` within this block of code to make it more meaningful and understandable.  

One way to do this is to use the substitute command repeatedly, but there are several faster ways.

If there are only a few lines and you can count the size of the scope within which you want to search and replace, you place your cursor at the beginning of the method and issue the command `:.,+4s/time/start_time`.  Here `.` refers to the current line; `,` is a separator, `+4` refers to the next four lines.

Suppose your cursor is far away and you have the line number turned on.  Let say the method above appears at Lines 125 to 131.  You can issue the command `:125,131s/time/start_time``.

Alternatively, you can use VISUAL-LINE mode.  Place the cursor at the beginning of the method, and press SHIFT-V.  This enters the VISUAL-LINE mode.  Now, navigate to select the scope within which you want to search and replace (`5j` or `}` works in this case), and press `:`.  You will see that the command prompt is pre-filled with `:'<,'>` to signify the selected range.  Continue typing `s/time/start_time` and ENTER.

[^1]: If this does not work, it means that your terminal or OS is intercepting the shortcut keys for other purposes.  For instance, CTRL-V might be interpreted as "paste" in Windows.  You need to remap the hotkeys either in Vim or your app/OS.

### Changing all occurrences in a file

Let's say that you have a typo in a file, where you have named all variables `angel` instead of `angle`, and you want to fix all occurrences of this in the file.  You can use `%` to signify that the range of substitution is the entire file.

The command `:%s/angle/angle/g` should replace all occurances for you.

## Typing Long Variable/Function Names

It is a good habit to give meaningful names to the variables, methods, and types in our programs.  To avoid cryptic names such as `noc`, it is recommended that we use English words, such as `num_of_customers`.  Such names can get very long.  Having a long name has several disadvantages.  First, it takes more keystrokes to type.  Second, it increases the likelihood of typos. 

Here are two useful tricks that can save you from typing long names.

### Auto-completion

You can type CTRL-P or CTRL-N in {--NORMAL--} INSERT mode to auto-complete a word.  So you only need to type the long name the first time.  Subsequently, type the prefix and auto-complete.

### Abbreviation

You can set up a temporary abbreviation in your `~/.vimrc`.  Example
```
ab noc num_of_customers
```

After the configuration is read, you only need to type `noc` in your code and it will be automatically expanded to `num_of_customers`.

This is useful for functions from the CS1010 library, such as `cs1010_println_long`, as well.

## Compiling without Leaving Vim

During development, we go through many iterations of the edit-compile-test cycles.  It would save time if we could do so without leaving Vim.

### TERMINAL mode

It is often useful to split your Vim window to open up a terminal, by the `:terminal` command.  From the terminal, you can run `make` to compile and run your code.

### Invoking Shell Commands

You can use `:!` to run a shell command without the terminal.  So `:!ls *.c` would let list all C files in the current directory without leaving Vim.

### Invoking Make

Unix tools are designed to work together.  `make` and `vim` are friends.  You can use `:make` to invoke `make` directly from within `vim`.

## Fixing Mistakes

### Undo and Redo

`u` undoes the last action.  Ctrl-R redoes the action. 

A related command, `.`, repeats the last action.

!!! Tips "&lt;count&gt;-&lt;movement&gt;" 
    Instead of repeating the commands above $k$ times, you can issue the command $k$ followed by the command.  For instance, `6u` would undo six times.  `4.` would repeat the last command 4 times.


### Backup Files

In case, you accidentally wrote over your precious code, or messed up in other ways, the last saved copy of your files can be found under `~/.backup`.  You can copy it back out.

### Swap Files

Vim automatically saves the files you are editing into temporary swap files, with the extension .swp. These files are hidden, so you don't normally see them when you run ls. (You need to run ls -a to view the hidden files)

The swap files are useful if your editing session is disrupted before you save (e.g., the network is disconnected, you accidentally close the terminal, your OS crashes, etc).

When you launch vim to edit a file, say, `echo.c`. vim will check if a swap file `.echo.c.swp` exists. If it does, vim with display the following

```
Found a swap file by the name ".echo.c.swp"
          owned by: elsa   dated: Sat Aug 21 15:01:04 2021
         file name: ~elsa/echo.c
          modified: no
         user name: elsa   host name: pe116
        process ID: 7863 (STILL RUNNING)
While opening file "echo.c"
             dated: Mon Jul 12 18:38:37 2024

(1) Another program may be editing the same file.  If this is the case,
    be careful not to end up with two different instances of the same
    file when making changes.  Quit, or continue with caution.
(2) An edit session for this file crashed.
    If this is the case, use ":recover" or "vim -r echo.c"
    to recover the changes (see ":help recovery").
    If you did this already, delete the swap file ".echo.c.swp"
    to avoid this message.

Swap file ".echo.c.swp" already exists!
[O]pen Read-Only, (E)dit anyway, (R)ecover, (Q)uit, (A)bort:
```

The messages above are self-explanatory. Read it carefully. Most of the time, you want to choose "R" to recover your edits, so that you can continue editing. Remember to remove the file `.echo.c.swp `after you have recovered. Otherwise, vim will prompt you with the above messages every time you edit `echo.c``.

!!! Warning

    If `echo.c` is newer than the state saved in `.echo.c.swp`, and you recover from `.echo.c.swp`, you will revert to the state of the file as saved in the swap file. This can happen if (i) you edited the file without recovery, or (ii) you recovered the file, and continued editing, but did not remove the `.echo.c.swp` file after.  The latest changes to `echo.c` would be lost.

