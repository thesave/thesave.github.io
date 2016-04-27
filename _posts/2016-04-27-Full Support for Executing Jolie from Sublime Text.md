---
layout: post
title:  "Full Support for Executing Jolie from Sublime Text"
date:   2016-04-26 10:00:00
categories: jolie sublime-text
---

[Version 1.2.2](https://github.com/thesave/sublime-Jolie/releases/tag/1.2.2) of the "jolie" plugin for the Sublime Text editor now supports the direct execution of Jolie programs from the editor's interface on the main operative systems, i.e., Windows, OSX, and Linux.

<img src="/imgs/sublime-jolie.jpg" alt="">

Executing a Jolie program is as easy as pressing Ctrl+B (or Cmd+B on OSX) keys, which is the shortcut for launching the build command in the editor (Tools > Build).

Currently, the Jolie programs execute in different terminals according to the OS and the installed terminal emulators:

- Windows: the default prompt console;
- OSX: the default Terminal app or [iTerm](https://www.iterm2.com/) in case it is installed;
- Linux: uses (and requires) `x-terminal-emulator`, which automatically start the OS-defined default terminal (e.g., `gnome-terminal` for Gnome, `konsole` for KDE, etc.).

---

Thanks to Enrico Benedetti, Matteo Sanfelici, and Stefano Pardini for their help in building and refining the launcher for Linux.