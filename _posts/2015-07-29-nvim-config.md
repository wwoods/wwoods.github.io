---
layout: post
title: Using Neovim (nvim) as an IDE
tags: neovim nvim vim vim-config
eye_catch: http://neovim.io/images/logo@2x.png
---

[Neovim](http://neovim.io) is a pretty cool successor to Vim, focusing on compatibility while adding asynchronous plugin functionality and trying to clean up the code base.  Having been fed up at various times with both Sublime Text (2 and 3) and Atom, and after realizing how much development I do over SSH, it seemed reasonable to check out using `vim` (or `nvim`, in this case) as my IDE.  The advantages essentially boil down to:

* Consistent IDE over SSH or locally,
* No need to ever use the mouse while coding (takes longer to learn, but is faster; this is aided by [vim-easymotion](https://github.com/easymotion/vim-easymotion) and [Ctrl-P](https://github.com/kien/ctrlp.vim)),
* It's been around forever, and frankly, probably always will be.

Some work really benefits from visual (read: mouse) input, but I have never found that to be the case while programming.

<!--more-->

So, I present my `.nvimrc` file, pieced together using [Vundle](https://github.com/gmarik/Vundle.vim).  Vundle has its own install instructions.  Aside from the filesystem changes, all of the `.nvimrc` changes necessary are reflected in the file below.  For `nvim`, one small change was needed to the instructions for `vim` (reflected in the `.nvimrc` file below); `vundle#begin()` requires an argument to redirect for `nvim`-specific settings: `vundle#begin('~/.nvim/bundle')`.  For everything else, simply copying the below file as `~/.nvimrc`, and running `nvim +PluginInstall +q` will be sufficient to set up the IDE.  It's that easy!

```vim
""""""" Plugin management stuff """""""
set nocompatible
filetype off

set rtp+=~/.nvim/bundle/Vundle.vim
call vundle#begin('~/.nvim/bundle')

Plugin 'gmarik/Vundle.vim'

" Custom plugins...
" EasyMotion - Allows <leader><leader>(b|e) to jump to (b)eginning or (end)
" of words.
Plugin 'easymotion/vim-easymotion'
" Ctrl-P - Fuzzy file search
Plugin 'kien/ctrlp.vim'
" Neomake build tool (mapped below to <c-b>)
Plugin 'benekastah/neomake'
" Autocomplete for python
Plugin 'davidhalter/jedi-vim'
" Remove extraneous whitespace when edit mode is exited
Plugin 'thirtythreeforty/lessspace.vim'

" Screen splitter.  Cool, but doesn't work with nvim.
"Plugin 'ervandew/screen'

" LaTeX editing
Plugin 'LaTeX-Box-Team/LaTeX-Box'

" Status bar mods
Plugin 'bling/vim-airline'
Plugin 'airblade/vim-gitgutter'

" Tab completion
Plugin 'ervandew/supertab'


" After all plugins...
call vundle#end()
filetype plugin indent on

""""""" Jedi-VIM """""""
" Don't mess up undo history
let g:jedi#show_call_signatures = "0"


""""""" SuperTab configuration """""""
"let g:SuperTabDefaultCompletionType = "<c-x><c-u>"
function! Completefunc(findstart, base)
    return "\<c-x>\<c-p>"
endfunction

"call SuperTabChain(Completefunc, '<c-n>')

"let g:SuperTabCompletionContexts = ['g:ContextText2']


""""""" General coding stuff """""""
" Highlight 80th column
set colorcolumn=80
" Always show status bar
set laststatus=2
" Let plugins show effects after 500ms, not 4s
set updatetime=500
" Disable mouse click to go to position
set mouse-=a


""""""" Python stuff """""""
syntax enable
set number showmatch
set shiftwidth=4 tabstop=4 softtabstop=4 expandtab autoindent
let python_highlight_all = 1

""""""" Keybindings """""""
" Set up leaders
let mapleader=","
let maplocalleader="\\"

" Mac OS X option-left / right
noremap â b
noremap æ e
inoremap â <C-o>b
inoremap æ <C-o>e<right>
" Note - this required binding in preferences (Cmd-,) option+backspace to
" escape+z.
" Why this one is complicated - <C-o> at end of line moves cursor by one
" character, which means a trailing character could be left.
inoremap <expr> ú col('.')>1 ? 'T<Left><C-o>db<Delete>' : '<Backspace>T<Left><c-o>db<Delete>'
" Requires binding option+forward delete to escape
inoremap ø <C-o>dw

" Arrow keys up/down move visually up and down rather than by whole lines.  In
" other words, wrapped lines will take longer to scroll through, but better
" control in long bodies of text.
" NOTE - Disabled since <leader><leader>w|e|b works well with easymotion
"noremap <up> gk
"noremap <down> gj

" Neomake and other build commands (ctrl-b)
nnoremap <C-b> :w<cr>:Neomake<cr>

autocmd BufNewFile,BufRead *.tex,*.bib noremap <buffer> <C-b> :w<cr>:new<bar>r !make<cr>:setlocal buftype=nofile<cr>:setlocal bufhidden=hide<cr>:setlocal noswapfile<cr>
autocmd BufNewFile,BufRead *.tex,*.bib imap <buffer> <C-b> <Esc><C-b>
```

