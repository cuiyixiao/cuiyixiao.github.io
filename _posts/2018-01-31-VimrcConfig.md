```
set nocompatible
filetype off
syntax enable
set number
set backspace=indent,eol,start
set foldmethod=syntax
set foldlevel=100
set helplang=cn
set encoding=utf-8
set number
set history=1000
set autoindent
set smartindent
set expandtab
set tabstop=4
set softtabstop=0
set shiftwidth=4
set smarttab
set showmatch
set vb t_vb=
set ruler
set incsearch
set hlsearch
set nobackup
set termencoding=utf-8
set fileencoding=utf-8
set fileencodings=utf-8
set fileformat=unix
set fileformats=unix
set pastetoggle=<F9>
set mouse=h
set clipboard=unnamed

map <C-A> ggVG <F2>
map <F2> "+y
map <F5> :w <CR>:!g++ -Wall % -o %< && ./%< <CR>

if has("autocmd")
    au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
endif
nmap <F5> ：call CompileRunGcc()<CR>
func! CompileRunGcc()
    exec "w"
    exec "!g++ % -0 %<"
    exec "! ./%<"
endfunc
```


- vimrc配置
