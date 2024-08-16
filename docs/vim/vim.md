---
title: vim
layout: default
nav_order: 2
has_children: false
permalink: docs/vim
has_toc: false
---

## 快速入门

可以通过`vimtutor`命令快速入门，如果你的vimtutor不是中文可以点击[vimtutor中文](/assets/file/vimtutor)下载中文版本

## 个人vim配置

[vim-plug](https://github.com/junegunn/vim-plug)插件管理  

{: .highlight }
这里两行`autocmd ColorScheme`配置特别有意思会让光标所在行加粗和斜体，别横向好用更优雅

```shell
colorscheme  murphy
set cursorline
autocmd ColorScheme * highlight! Cursorline cterm=bold,italic ctermbg=238 guibg=Grey90 
autocmd ColorScheme * highlight! CursorLineNr cterm=bold,italic ctermfg=165 ctermbg=238 guibg=Grey90 
syntax on
set scrolloff=6
set number
set wrap
set showcmd
set wildmenu
set hlsearch
hi Search ctermbg=LightBlue
hi Search ctermfg=Red
set incsearch
set ignorecase
set smartcase
map s :w<CR>
map qq :q<CR>

call plug#begin()
Plug  'vim-airline/vim-airline'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
call plug#end()
```

