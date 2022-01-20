## Environment
![Ubuntu](https://img.shields.io/badge/Ubuntu-16.04-orange)
https://img.shields.io/badge/Ubuntu-16.04-orange
https://img.shields.io/badge/Vim-7.4-green

## Set up
1. git clone https://github.com/VundleVim/Vundle.vim.git~/.vim/bundle/Vundle.vim

2. Copy and paste under script to ~/.vimrc

~~~
set nocompatible     
filetype off     
set rtp+=~/.vim/bundle/Vundle.vim     
call vundle#begin()     
Plugin 'VundleVim/Vundle.vim'     
Plugin 'Yggdroot/indentLine'     
call vundle#end()            
filetype plugin indent on     

let g:indentLine_char = '┆'     
let g:indentLine_color_term = 'darkgrey'     
let g:indentLine_color_gui = 'darkgrey'     
let g:indentLine_leadingSpaceChar = '·'     
let g:indentLine_leadingSpaceEnabled  = 1     
~~~


3. Run :PluginInstall
