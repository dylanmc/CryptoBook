# Implementing More Complex Programs

At this point, you've written some Cryptol at the command-line, but if
you had to leave your machine, reboot, or whatever, you had to start
from scratch. "That's no good", I hear you say, "there must be 
another way!" Indeed there is.

## Storing Cryptol programs in files

In addition to typing commands at the Cryptol interpreter, Cryptol can
load files that have programs in them. Programs are slightly different
from the commands you've been typing so far. The main difference is
that you don't need `let` when you're defining a variable (like
`alphabet`) or a function (like `encrypt').

The next thing you'll want to do is create a working directory (or
folder) to keep your project files for this book.
You'll need to figure out how to edit _text files_ on your computer.
Text files are different from, say, word processing documents, in that
they do not have any formatting (no **bold** or _italics_, for
example). They're literally just sequences of ASCII characters (or
UTF-8 if you have a fancy editor). The command-line programs `vim` and
`emacs` are still popular, even though they look like Wargames-era
technology^[no coincidence, either: the original versions of these
programs were written almost 10 years
before Wargames came out. They're still popular because they're very
powerful, but that's a lesson for another book.]. More modern text
editors include _Sublime Text_ (`https://www.sublimetext.com/`),
_nano_ (`https://www.nano-editor.org/`), and many others. If you're on
Windows, and are in a pinch, both _Notepad_ and _Wordpad_ can "Save
As" _plain text_.

## Exercise: rewriting our Caesar Cipher program

Start this exercise by creating your project directory. On a Unix
systems (like Linux or MacOS), you'll want to start up a Terminal
window, on Windows start a shell window (type `cmd` in the search bar),
and do something like this:
`$ ` **`mkdir CryptoBook`**  
`$ ` **`cd CryptoBook`**  

Then, using whichever editor you choose, create a file called
`caesar.cry` inside that directory. It should contain the following:

```
// Caesar Cipher
alphabet = ['a' .. 'z']
toLower c = if c < 0x61 then c + 0x20 else c
asciiToIndex c = (toLower c) - 'a'
encryptChar wheel c = if c == ' '
    then c
    else wheel @ (asciiToIndex c)
codeWheel key = reverse alphabet >>> key
encrypt key message =
    [ encryptChar (codeWheel key) c | c <- message ]
decrypt key message = encrypt key message
```

The first line (`// Caesar Cipher`) is a comment: it's text to remind
you what's in the file. With our more complex programs, we'll add more
comments. They'll help remind us what's going on.

## Running Cryptol with `caesar.cry`

Now that we have a file with our Caesar Cipher in it. To run
that program, launch Cryptol from inside your project directory.

`$ ` **`cryptol caesar.cry`**  
```
                       _        _
  ___ _ __ _   _ _ __ | |_ ___ | |
 / __| '__| | | | '_ \| __/ _ \| |
| (__| |  | |_| | |_) | || (_) | |
 \___|_|   \__, | .__/ \__\___/|_|
           |___/|_|  version 2.4.0

Loading module Cryptol
Loading module Main
```  
`Main> ` **`encrypt 11 "using files now"`**  
```
[warning] at <interactive>:1:1--1:28:
  Defaulting type parameter 'bits'
             of literal or demoted expression
             at <interactive>:1:9--1:10
  to 3
[0x6a, 0x6c, 0x76, 0x71, 0x78, 0x20, 0x79, 0x76, 0x73, 0x7a, 0x6c,
 0x20, 0x71, 0x70, 0x68]
```

Whoops, we still have to turn on ASCII output:^[Skipping the warning again.]
If you got `[error] can't find file: caesar.cry` instead, then you
need to use a shell command to move into your CryptoBook directory.
If that's proving difficult, ask a partner or look up _command line
navigating files_ for the operating system you're running on.

`Main> ` **`:set ascii=on`**  
`Main> ` **`encrypt 4 "using files now"`**  
`"jlvqx yvszl qph"`

