# Using nvim

ref: https://x-team.com/blog/neovim-flutter/

## Plugins

    call plug#begin('~/.vim/plugged')

    "Dart/Flutter
    Plug 'dart-lang/dart-vim-plugin'
    Plug 'thosakwe/vim-flutter'
    Plug 'natebosch/vim-lsc'
    Plug 'natebosch/vim-lsc-dart'

    call plug#end()



Running emulator

    :CocCommand flutter.emulators

## Important Flutter Commands

    :FlutterRun <args>: calls Flutter Run <args>
    :FlutterHotReload: triggers a hot reload on the Flutter process
    :FlutterHotRestart: triggers a hot restart on the Flutter process
    :FlutterQuit: quits the current Flutter process
    :FlutterDevices: opens a new buffer and writes the output of Flutter devices to it
    :FlutterSplit: opens Flutter output in a horizontal split
    :FlutterEmulators: executes a Flutter Emulator process
    :FlutterEmulatorsLaunch: executes a Flutter Emulator --launch process with any provided arguments
    :FlutterVisualDebug: toggles visual debugging in the running Flutter process
