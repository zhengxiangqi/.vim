# Vim配置
配置Vim编辑器，使用Monokai主题，支持语法高亮、目录索引、tab标签、ctag、插件管理等。

**克隆.vim文件夹**
```bash
cd ~
git clone git@github.com:zhengxiangqi/.vim.git
```

**安装插件**
```bash
# 克隆插件管理工具
git clone http://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
# 打开Vim
vi
# 输入冒号进入命令模式，并执行以下指令安装插件，安装完成重新启动Vim即可正常使用
BundleInstall
# 后续要安装更多插件请查看Bundle文档：https://github.com/VundleVim/Vundle.vim
```

**创建.vimrc文件**
```bash
cd ~
touch .vimrc
# 使用文本编辑器打开.vimrc文件，并填入配置信息，例如使用SublimeText
subl .vimrc
```

.vimrc文件内容如下
```text
" Author: zhengxiangqi<1075953039@qq.com>
" Description: zhengxiangqi's personal vim config file
" Blog: https://zhengxiangqi.github.io
" Since:2015-07-01
" History:
" 2013-11-18
" +update vim7.4 and add the new font 'Yahei_Consolas_hybrid'
" 2012-10-28
" +compress vimrc
" 2012-06-08
" +add Neobundle to manage plugins
" 2011-09-08
" +build vimrG
" ============
" Environment
" ============
syntax on
" 颜色主题
colorscheme monokai
" 自动补全不显示相关条目详情
set completeopt=longest,menu
" 关闭vim兼容模式
set nocp
" 允许对文件类型检测
filetype plugin on
" 行控制
set noscrollbind
set linebreak
set nocompatible
set textwidth=160
set wrap
" 标签
set tabpagemax=9
set showtabline=2
" 行号和标尺
set number
set ruler
" 命令行于状态行
set ch=2
set ls=2 " 始终显示状态行
set wildmenu "命令行补全以增强模式运行
" Search Option
set hlsearch  " Highlight search things
set magic     " Set magic on, for regular expressions
set showmatch " Show matching bracets when text indicator is over them
set mat=2     " How many tenths of a second to blink
set noincsearch
" 制表符
set tabstop=4
set expandtab
set smarttab
set shiftwidth=4
set softtabstop=4
set iskeyword+=_,$,@,%,#,-
" 状态栏显示目前所执行的指令
set showcmd 
" 缩进
set autoindent
set smartindent
" 自动重新读入
set autoread
" 插入模式下使用 <BS>、<Del> <C-W> <C-U>
set backspace=indent,eol,start
" 设定在任何模式下鼠标都可用
set mouse=a
" 自动改变当前目录
set autochdir
" 备份和缓存
set nobackup
set noswapfile
" 代码折叠
set foldmethod=marker
" 共享外部剪贴板
set clipboard+=unnamed
" Diff 模式的时候鼠标同步滚动 for Vim7.3
set cursorbind
" 设置ctags引用的文件路径
set tags+=~/.vim/tags/pytags
" 定义 <Leader> 为逗号
let mapleader = ","
let maplocalleader = ",""设置快速编辑.vimrc文件 ,e 编辑.vimrc
" 快速修改 vimrc 文件
if has("win32")
    map <silent> <leader>e :e $VIM/_vimrc<cr>
else
    map <silent> <leader>e :e $VIM/.vimrc<cr>
endif
"保存.vimrc文件后会自动调用新的.vimrc
autocmd! bufwritepost _vimrc source $VIM/_vimrc
 
" =====================================================================
" Multi_language setting default encoding UTF  
" =====================================================================
if has("multi_byte")
    set encoding=utf-8
    if has('win32')
        language chinese
        let &termencoding=&encoding
    endif
    set nobomb " 不使用 Unicode 签名
    if v:lang =~? '^\(zh\)\|\(ja\)\|\(ko\)'
        set ambiwidth=double
    endif
else
    echoerr "Sorry, this version of (g)vim was not compiled with +multi_byte"
endif
 
" ===========================================
" AutoCmd
" ===========================================
if has("autocmd")
    filetype plugin indent on
    " 括号自动补全
    func! AutoClose()
        :inoremap ( ()<ESC>i
        :inoremap " ""<ESC>i
        :inoremap ' ''<ESC>i
        :inoremap { {}<ESC>i
        :inoremap [ []<ESC>i
        :inoremap ) <c-r>=ClosePair(')')<CR>
        :inoremap } <c-r>=ClosePair('}')<CR>
        :inoremap ] <c-r>=ClosePair(']')<CR>
    endf
    func! ClosePair(char)
        if getline('.')[col('.') - 1] == a:char
            return "\<Right>"
        else
            return a:char
        endif
    endf
    augroup vimrcEx
        au!
        autocmd FileType text setlocal textwidth=160
        autocmd BufReadPost *
                    \ if line("'\"") > 0 && line("'\"") <= line("$") |
                    \   exe "normal g`\"" |
                    \ endif
    augroup END
    " Auto close quotation marks for PHP, Javascript, etc, file
    au FileType php,javascript,c,cpp exe AutoClose()
endif

" ==================
" script's functions  
"===================
" 获取当前目录
func! GetPWD()
    return substitute(getcwd(), "", "", "g")
endf
" 全选
func! SelectAll()
    let s:current = line('.')
    exe "norm gg" . (&slm == "" ? "VG" : "gH\<C-O>G")
endfunc
"函数后面加上！是防止vimrc文件重新载入时报错
"实现光标位置自动交换:) -->  ):
function! Swap()
    if getline('.')[col('.') - 1=~ ")"
        return "\<ESC>la:"
    else
        return ":"
    endif
endf
"------------------------------------------------------------------------------------
"SwitchToBuf()实现它在所有标签页的窗口中查找指定的文件名，如果找到这样一个窗口，
"就跳到此窗口中；否则，它新建一个标签页来打开vimrc文件
"上面自动编辑.vimrc文件用到的函数
function! SwitchToBuf(filename)
    let bufwinnr = bufwinnr(a:filename)
    if bufwinnr != -1
        exec bufwinnr . "wincmd w"
        return
    else
        " find in each tab
        tabfirst
        let tab = 1
        while tab <= tabpagenr("$")
            let bufwinnr = bufwinnr(a:filename)
            if bufwinnr != -1
                exec "normal " . tab . "gt"
                exec bufwinnr . "wincmd w"
                return
            endif
            tabnext
            let tab = tab + 1
        endwhile
        " not exist, new tab
        exec "tabnew " . a:filename
    endif
endfunction

" =============
" Key Shortcut
" =============
nmap <leader>l :set list!<cr>
for i in range(1, &tabpagemax)
    exec 'nmap <A-'.i.'> '.i.'gt'
# .vim
Vim配置文件夹
endfor
set sessionoptions=buffers,curdir,resize,folds,tabpages,winsize,buffers
autocmd VimLeave * mks! $VIM/sessions/Session.vim  
nmap <F2> :wa<Bar>exe "mksession! " . v:this_session<CR>:so $VIM\sessions\session.vim<cr>
map <F6> zM    " 关闭所有折叠
map <F7> zR    " 打开所有折叠
"map <F10> :!ctags -R<CR><CR>
map <F10> :!ctags -I __THROW --file-scope=yes --langmap=c:+.h --languages=python --links=yes --c-kinds=+p --fields=+S  -R -f ~/.vim/tags/pytags ~/workspaces/Workspaces/python/improcess/<CR><CR>
"map <C-F10> :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR>
map <silent><leader>ww <c-w>w
map <silent><leader>wl <c-w>l
map <silent><leader>wk <c-w>k
map <silent><leader>wj <c-w>j
map <silent><leader>wh <c-w>h
map <silent><leader>wn <c-w>n
map <silent><leader>wr <c-w>r
map <silent><leader>wR <c-w>R
map <silent><leader>wx <c-w>x
map <silent><leader>wo <c-w>o
imap jk <esc>
" 新增vim-debug插件快捷键
map <D-F5> :Dbg into<CR>
map <D-F6> :Dbg over<CR>
map <D-F7> :Dbg out<CR>
map <D-F8> :Dbg run<CR>
map <D-F9> :Dbg break<CR>
map <D-F10> :Dbg eval<CR>
map <D-F11> :Dbg .<CR>
map <D-F12> :Dbg quit<CR>
" vi代码补全提示快捷键
inoremap <C-F>  <C-X><C-F>
inoremap <C-L>  <C-X><C-L>
" 保存快捷键，保存的同时将生成ctags
map <C-G> :w<CR><F10>
"==================================================================

"=====================
" vim-powerline插件
"=====================
set laststatus=2
set t_Co=256
let g:Powerline_symbols = 'unicode'
set encoding=utf-8

"=====================
" taglist 插件
"=====================
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
let Tlist_Process_File_Always=1

"=====================
" winmanager插件
"=====================
let g:winManagerWindowLayout='FileExplorer|TagList'
nmap wm :WMToggle<cr>
let g:persistentBehaviour=0

"=====================
" MiniBufferExplorer插件
"=====================
let g:miniBufExplMapCTabSwitchBufs=1
let g:miniBufExplMapWindowsNavVim=1
let g:miniBufExplMapWindowNavArrows=1
let g:miniBufExplorerMoreThanOne=0

"=====================
" pydiction插件
"=====================
let g:pydiction_location = '~/.vim/bundle/pydiction/complete-dict'
autocmd FileType py set shiftwidth=4 | set expandtab

"=====================
" jedi-vim插件
"=====================
let g:jedi#auto_initialization = 0
"let g:jedi#use_tabs_not_buffers = 1
"let g:jedi#completions_enabled = 0

"=====================
" syntastic插件
"=====================
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*
let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0

"=====================
" vundle VIM插件管理器
"===================== 
" 安装方式：git clone http://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
filetype off
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" let Vundle manage Vundle
" required!
Bundle 'gmarik/Vundle'
" 可以通过以下四种方式指定插件的来源
" a) 指定Github中vim-scripts仓库中的插件，直接指定插件名称即可，插件名中的空格使用“-”代替。
"Bundle 'L9'

" b) 指定Github中其他用户仓库的插件，使用“用户名/插件名称”的方式指定
" Bundle 'tpope/vim-fugitive'
" Bundle 'Lokaltog/vim-easymotion'
" Bundle 'rstacruz/sparkup', {'rtp': 'vim/'}
" Bundle 'tpope/vim-rails.git'

" c) 指定非Github的Git仓库的插件，需要使用git地址
" Bundle 'git://git.wincent.com/command-t.git'

" d) 指定本地Git仓库中的插件
" Bundle 'file:///Users/gmarik/path/to/plugin'

"指定需要安装或卸载的插件
Bundle 'https://github.com/Lokaltog/vim-powerline.git'
Bundle 'taglist.vim'
Bundle 'winmanager'
Bundle 'minibufexpl.vim'
Bundle 'pydiction'
Bundle 'pyflakes'
"Bundle 'vim-debug'
Bundle 'davidhalter/jedi-vim'
Bundle 'git@github.com:drmingdrmer/xptemplate.git'
Bundle 'vim-syntastic/syntastic'

filetype plugin indent on   " required!
"==============================================================================================
```

