# openwith-fzf
Very simple, fast and reliable bash script to display an "openwith" menu with fzf.
I can also be use to get the default application associated with a file type, and so replace xdg-open, rifle or any 'file opener' program.

This script been designed with the intention of providing an "open with" kind of menu in contexts like [`ranger`](https://github.com/ranger/ranger) or [`fzf-nova`](https://github.com/gotbletu/shownotes/tree/master/fzf_nova). It is just a shell command which takes a list of file paths as arguments and displays a list of applications able to open these files in [`fzf`](https://github.com/junegunn/fzf). Then, it is possible to select in fzf the opening application and the script opens the files with this application.

The script shared here is part of my own config, so obviously, you should edit it to fit the applications installed and used on your desktop... As it just an "amateur" creation, tested only on my computer, you can't expect it will run in any environment.
Dependencies : [`fzf`](https://github.com/junegunn/fzf), [`file`](https://man.archlinux.org/man/file.1) from base-devel package (should be already installed on your computer). By default, but optionnal, another script of mine : openrule.

## Use

If you copy the script to /usr/local/bin or /usr/bin, you can use it like any application, as a shell command, from the terminal, in a script...
It takes as only arguments a list of files, into quotes if the name contains spaces. The only flg, -d, echo the default application associated with the file(s)
passed as arguments, without opening the menu. So it can be use in combinaison with other scripts:
  - `$(openwith -d test.sh) test.sh` will open test.sh with the default application you defined, nano, vim, emacs...
  - `openrule d $(openwith -d toto.pdf) toto.pdf` will open toto.pdf according to the rules defined in openrule, with the default application given by openwith.
 With the -d flag, if several files are passed as arguments and they are not of the same type, openwith output as first argument '1', you can use this behavior in your scripts...
  - `openwith -d toto.pdf tata.jpg tutu.jpg` will output: 1 zathura nomacs nomacs
 It can take directories as arguments. It will not open them, but this can be useful for example with [dragon](https://github.com/mwh/dragon), which can drag and drop directories.

## Manage what application is associated with a file type

The main function of the script checks the file extension ; if no extension is found, it performs (if maxdeepcheck > 0) a 'file' command getting the file type of the sublist of files without known extensions. Both functions set the same variables : each application is represented by a variable. If the app variable is set to 1, the application will be added to the menu by the 'applist' function. So, to assignate to a file type an application, you have to add a variable specific to this app, and in the appfunction, if your variable have a certain value (1 is ok), append the applist list with the name of the application terminated by \n. the default application is set by `echo` it.

Example:
```bash
### SECTION WHERE YOU DEFINE THE 'CATEGORIES' ###
pdf='em=1 && za=1 && echo zathura'
...
_extcheck(){
...
  elif [[ $file == *.pdf ]];then   eval $pdf
...
_applist(){
...
[[ $za = 1 ]] && applist+="zathura\n"
[[ $em = 1 ]] && applist+="emacs\n"
...
_deepcheck(){}
...
case $ftype in
  ...
  pdf) eval $pdf
```
Set zathura and emacs as potential applications opening pdf files, and zathura as the default one -d returns.
It if better to set the most file type on which you will use the openwith command near the top of the elif conditions in extcheck, as the for loop don't execute the other elif - and so the script runs faster... if it was necessary (we talk about gaining thousandth of seconds...when hundreds of files are scanned)... In any case, it is better to add ext checks in extcheck than to use the deepcheck function.

## Configuration

It is a very simple script, so you probably can edit it to customize it yourself. You can also use the 'build-in' variables to ajdust some settings:
- ```maxargs``` limit the number of arguments the script will check (=the max limit of files to open), 100 by default.
- ```maxdeepcheck``` does the same, but for the sub function 'deepcheck' which tries to determine the file type using the linux 'file' command if the files have no extension known, 50 by default (takes less than 0.2 sec to scan 50 files).
- ```alphsort``` : sort the application list in an alphabetic number.
- ```warningmsg``` and ```filescount``` : enable/disable the 1) warning messages if a directory is selected or if several (and potentially incompatible) file types are selected, 2) the number of files taked as arguments.

## Hack tips

The deepcheck is MUCH slower than the main execution function extcheck. The later use only a simple bash comparison, which is REALLy fast : even with many elif condition checks, it takes less than 0.05 sec for hundreds or even thousands of files. But the 'file' command is obviously slower, as it is executed for each file, that's why it is usefull to set the maxdeepcheck variable to limit the number of files on which a "deep check" will be perform. Use the grep command in the extcheck function will be also much slower than the bash comparison.

Take note than the mime type is not so reliable. Many rare and special file types are not well categorized with mime types : for example, many rare image formats are not part of the "image" mime type (x3f will be inode/x-empty, hdr and ras are both application/octet-stream, etc.) and some different format can share the same mime type... So the approach by file extension is in fact quite happy. All mime types can be found normally in /usr/share/applications/mimeinfo.cache.

## Run it from ranger

In ranger, run :shell openwith %p. You can map this command to a key shortcut in rc.conf.

## Install

Just copy this script to /usr/bin/local (or /usr/bin, or use it as a script from anywhere else).
