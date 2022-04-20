# openwith-fzf
Very simple, fast and reliable bash script to display an "openwith" menu with fzf.

This script been designed with the intention of providing an "open with" kind of menu in contexts like [`ranger`](https://github.com/ranger/ranger) or [`fzf-nova`](https://github.com/gotbletu/shownotes/tree/master/fzf_nova). It is just a shell command which takes a list of file paths as arguments and displays a list of applications able to open these files in [`fzf`](https://github.com/junegunn/fzf). Then, it is possible to select in fzf the opening application and the script opens the files with this application.

The script shared here is part of my own config, so obviously, you should edit it to fit the applications installed and used on your desktop... As it just an "amateur" creation, tested only on my computer, you can't expect it will run in any environment.
Dependencies : [`fzf`](https://github.com/junegunn/fzf), [`file`](https://man.archlinux.org/man/file.1) from base-devel.

## Use

If you copy the script to /usr/local/bin or /usr/bin, you can use it like any application, as a shell command, from the terminal, in a script...
It takes as only argument a list of files, into quotes if the name contains spaces.

## Manage what application is associated with a file type

The main function of the script checks the file extension ; if no extension is found, it performs (if maxdeepcheck > 0) a 'file' command getting the file type of the sublist of files without known extensions. Both functions set the same variables : each application is represented by a variable. If the app variable is set to 1, the application will be added to the menu by the 'applist' function. So, to assignate to a file type an application, you have to add a variable specific to this app, and in the appfunction, if your variable have a certain value (1 is ok), append the applist list with the name of the application terminated by \n.

Example:
```bash
_extcheck(){
...
  elif [[ $file == *.pdf ]];then   em=1 && za=1
...
_applist(){
...
[[ $za = 1 ]] && applist+="zathura\n"
[[ $em = 1 ]] && applist+="emacs\n"
...
_deepcheck(){}
...
  PDF) em=1 && za=1
```
Set zathura and emacs as potential applications opening pdf files.

## Configuration

It is a very simple script, so you probably can edit it to customize it yourself. You can also use the 'build-in' variables to ajdust some settings:
- ```maxargs``` limit the number of arguments the script will check (=the max limit of files to open), 100 by default.
- ```maxdeepcheck``` does the same, but for the sub function 'deepcheck' which tries to determine the file type using the linux 'file' command if the files have no extension known.
- ```alphsort``` : sort the application list in an alphabetic number.
- ```warningmsg``` and ```filescount``` : enable/disable the 1) warning messages if a directory is selected or if several (and potentially incompatible) file types are selected, 2) the number of files taked as arguments.

## Hack tips

The deepcheck is MUCH slower than the main execution function extcheck. The later use only a simple bash comparison, which is REALLy fast, even when scanning hundreds of files. But the 'file' command is obviously slower, that's why it is usefull to set the maxdeepcheck variable to limit the number of files on which a "deep check" will be perform. Use the grep command will be even slower.

## Run it from ranger

In ranger, run :shell openwith %p. You can map this command to a key shortcut in rc.conf.

## Install

Just copy this script to /usr/bin/local (or /usr/bin, or use it as a script from anywhere else).
