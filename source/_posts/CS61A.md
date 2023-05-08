---
title: CS61A (Fall2022)
date: 2023-05-08 21:21:36
tags: 
- Course
- Python
- Scheme
categories: Course
---

A note about Berkeley.

# Lab00

## Setup

* Setup: WSL2 & WindowsTerminal
* Install python3: `sudo apt install python3`
* Install texteditor: `neovim`

## Using the terminal

* `~` represent the home directory.
* `echo $HOME$` to display the full PATH to your home directory.
* `pwd` to show the PATH of the current directory.

> Notices you are in the terminal or the python interpreter.

## Organize your files

* `ls` to list files in the current directory.
* `cd directoryname` to change into the sub-directory.
    * `cd ..` to change into the parent directory.
* `mkdir directoryname` to create a directory.
* `unzip lab00.zip` to extract starter files.
* `mv PATH1 PATH2` to move files.

## Python basics

* `python3` to enter the python interpreter.
* Primitive expressions: numbers and booleans, they just evaluate to themselves.
* Arithmetic expressions: `+`, `-`, `*`, `%`.
  `**` (exponentiation), `/`(Floating point division), `//`(Floor division).
* Strings: some characters wrapped in either `'` or `"`.
* Assignment statements: `=`.

## Do the assignments

* `python3 ok -q python-basics -u --local` under the directory `lab00`.
    * `--local` skip the email verification.
* Fix the code in `lab00.py`, and `python3 ok --local`.

## Useful python commandline options

* `-i`  run Python code line by line.
