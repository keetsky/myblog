---
title: Vim终极配置
date: 2018-02-10 20:03:19 
comments: true
english_title: VIM config   #对于中文标题需要个英文字体代替中文显示在URL上
categories:  #分类 
- ubuntu 
tags:	     #两个标签
- VIM

---

#### 图省事的配置
[github_web](https://github.com/sakurazhu/vim)

#### 经典配置
> spf13-vim +YouCompleteMe
> spf13-vim+neocomplete  (推荐)

1. spf13-vim 安装
```
$ curl https://j.mp/spf13-vim3 -L > spf13-vim.sh && sh spf13-vim.sh
```
2. YouCompleteMe 安装
[安装教程网址](https://www.bbsmax.com/A/LPdoMe6jJ3/)
  `2.1.1` 在spf13中指定安装YouCompleteMe插件也很方便，只需要在文件~/.vimrc.before.local中添加下面一行
```
let g:spf13_bundle_groups=['general', 'youcompleteme'] 
或 let g:spf13_bundle_groups['general', 'writing', 'neocomplcache', 'programming', 'php', 'ruby', 'python', 'javascript', 'html', 'misc', 'youcompleteme', ]
或 在.vimrc 中加入 Bundle 'Valloric/YouCompleteMe'
```
  然后采用Vundle命令安装
```
$ vim
在vim中输入 
:BundleInstall!
```
> 或`源码安装`
```
cd ~/.vim/bundle
git clone https://github.com/Valloric/YouCompleteMe.git
cd YouCompleteMe
git submodule update --init --recursive
./install.sh --clang-completer
```

`2.1.2` 编译ycm_core.so

```
cd ~/.vim/bundle/YouCompleteMe/
./install.py --all #编译支持所有功能
or
./install.py --clang-completer #只支持C/C++补全
```

`2.1.3` YouCompleteMe配置
   `.vimr`或`.vimrc.local`配置添加
[参考网址](http://howiefh.github.io/2015/05/22/vim-install-youcompleteme-plugin/)
[参考网址](https://zhuanlan.zhihu.com/p/33046090)
```
"
            let g:acp_enableAtStartup = 0
            " global conf which is needed to resolve name in system include
            " file or other  third-part include file
            let g:ycm_global_ycm_extra_conf = '~/.vim/.ycm_extra_conf.py'
            let g:ycm_server_python_interpreter='/usr/bin/python'
            " enable completion from tags
            let g:ycm_collect_identifiers_from_tags_files = 1
            let g:ycm_seed_identifiers_with_syntax = 1
            let g:ycm_confirm_extra_conf = 0
            let g:ycm_cache_omnifunc=0
            let g:ycm_key_invoke_completion = '<C-;>'
           " nnoremap <F5> :YcmForceCompileAndDiagnostics<CR>
            nnoremap <leader>jd :YcmCompleter GoToDefinitionElseDeclaration<CR>

 把nerdtree绑定到F5上
            let NERDTreeChDirMode = 2
            let NERDTreeWinSize = 30
            nmap <F5>  :NERDTreeToggle<CR> 

  缩进看起来一块一块的，比较难看，我习惯默认关闭它,可以通过<Leader>ig来开启
           let g:indent_guides_auto_colors = 0
   
   一般把tagbar绑在<F6>上
           nnoremap <F6> :TagbarToggle<CR> 
           let g:tagbar_autoclose=1 "在一个tag上按回车后，自动跳转到tag在文件的位置，并关闭tagbar
           
```
> `!`上面配置中全局.ycm_extra_conf.py路径很重要，如果不配置将无法解析C/C++头文件

.ycm_extra_conf.py 模版位于YouCompleteMe/third_party/ycmd/cpp/ycm/，其中-isystem flag用来配置系统头文件路径，-I用来配置第三方头文件路径, 一个支持C/C++工程的ycm_extra_conf.py部分配置文件修改添加如下：
```
'-std=c++11',
#'-std=c99',
# ...and the same thing goes for the magic -x option which specifies the
# language that the files to be compiled are written in. This is mostly
# relevant for c++ headers.
# For a C project, you would set this to 'c' instead of 'c++'.
'-x',
'c++',
'-isystem',
'../BoostParts',
#-isystem: system include file path按照自己的头文件添加iinclude路径
'-isystem', '/usr/include',
'-isystem', '/usr/local/include',
'-isystem', '/usr/include/c++/4.8.4',
'-isystem','/usr/include/x86_64-linux-gnu/c++',
]
```

----
2.2 基于vundle安装
[参考网址](https://www.cnblogs.com/274914765qq/p/4439189.html)
使用vundle进行安装，在`.vimrc`中添加如下代码
```
Bundle 'Valloric/YouCompleteMe'   "或直接在vim ：中输入 Bundle 'Valloric/YouCompleteMe'
```
保存退出后打开vim，在正常模式下输入
```
:BundleInstall  
```
等待vundle将YouCompleteMe安装完成，而后需要进行编译安装
```
cd ~/.vim/bundle/YouCompleteMe
./install.sh --clang-completer
```
在.vimrc中对YouCompleteMe的配置如下
```
" YouCompleteMe配置
let g:ycm_error_symbol = '>>'
let g:ycm_warning_symbol = '>*'
nnoremap <leader>gl :YcmCompleter GoToDeclaration<CR>
nnoremap <leader>gf :YcmCompleter GoToDefinition<CR>
nnoremap <leader>gg :YcmCompleter GoToDefinitionElseDeclaration<CR>
nmap <F4> :YcmDiags<CR>
```
nmap<C-a> :YcmCompleter FixIt<CR>  




##### 自动补全法2
因为安装`YouCompleteMe`太麻烦了，可能会安装`jedi-vim`代替;不过推荐使用neocomplete
在.vimrc 添加 jedi-vim 和 supertab
```
 call vundle#begin()  
...  
 Bundle 'davidhalter/jedi-vim'  
 Bundle 'ervandew/supertab'  
...  
 call vundle#end()            " required  
 filetype plugin indent on    " required  
```
 打开 vim 使用 :PluginInstall 命令安装插件
```
cd ~/.vim/bundle/jedi-vim/ && git submodule update --init
```
 在 .vimrc 添加
```
let g:SuperTabDefaultCompletionType = "context"  
let g:jedi#popup_on_dot = 0  
```
**`neocomplete安装`**
 `$vim --version`  查看是否有 +lua ,没有则需要 sudo apt-get install lua-devel 再重新安装vim
>  [下载网址](https://github.com/Shougo/neocomplete.vim])
   拷贝到～/.vim
   .vimrc添加  let g:neocomplcache_enable_at_startup = 1
  
  官网的Configuration Examples可以用

#### 问题
1. YouCompleteMe unavailable : requires Vim 7.4.143
   vim 版本不够新需要升级
   查看下vim版本`$vim --version`
```
sudo add-apt-repository ppa:jonathonf/vim 
sudo apt update 
sudo apt install vim 
#or
sudo add-apt-repository ppa:jonathonf/vim
sudo apt-get update && sudo apt-get upgrade
```
如果还不行，进行如下操作：sudo apt-get -u dist-upgrade（强制更新软件包到最新版本，并自动解决缺少的依赖包） ，问题解决。在不行就下载vim源码安装

2. ctags not found

在安装spf13集合包时，个人的ubuntu 系统并没有安装ctags，造成Tagbar插件未能成功安装。
所以在安装好ctags后，重新运行 BundleInstall，此时Tagbar插件才真正落户安家，发挥作用。

3. vim 有些卡
  vim -f
  gvim -f
#### 快捷键
[参考网址](http://conglang.github.io/2015/04/06/spf13-vim-cheat-sheet/)

1.  NERDTree

NERDTree 是一个文件浏览器。
```
Ctrl+e	打开/关闭NERDTree   或 <Leader>e方式     ,e
?	显示快速帮助
o或Ctrl+R	打开文件、目录和书签
go	打开选中文件，不过光标仍在NERDTree中
t	在新tab中打开选中节点/书签
T	功能与t相同，不过焦点仍在当前tab
i	在新split打开选中文件
gi	与i相同，不过光标仍在NERDTree中
s	在新vsplit中打开选中文件
gs	与s相同，不过光标仍在NERDTree中
O	打开选中目录所有子目录
x	关闭当前节点父节点
X	关闭当前节点所有子节点
D	删除当前书签
P	跳到根节点
p	跳到当前节点父节点
K	跳到本层级第一个节点
J	跳到本层级最后一个节点
C	设置选中目录为根结点
u	根结点向上跳出一级
U	与u相同，只是老根结点保持打开
r	刷新当前目录所有子目录
R	刷新当前根目录所有子目录
m	显示NERDTree的菜单
cd	将当前工作目录改为选中节点
CD	将根结点改为当前工作目录
f	切换是否打开文件过滤
F	切换是否显示文件
B	切换书签列表是否显示
A	最大最小化NERDTree窗口
:help NERDTree	NERDTree帮助手册
```
2.  EasyMotion
快速行跳转 
快速字跳转
```
,,w	当前光标后的所有word，提供快捷键跳转
,,b	当前光标前的所有word，提供快捷键跳转
,,j	当前光标后的所有行，提供快捷键跳转
,,k	当前光标后的所有行，提供快捷键跳转
```
3.  ctags
实现各种函数/变量跳转至各自声明处。
``` 
ctrl+]	当前光标处word，跳转至相同名称的函数处或者变量声明处
ctrl+t	跳转的返回
g]	当前光标处word，跳转至相同名称的函数处或者变量声明处，不同与ctrl+]，会列出所有相同名称的标签文件

,tt     打开tagbar界面
```
可手动设置快捷键：
```
nmap <F3>   <ctrl-T>  
nmap <F4>   <ctrl-]> 
```
> tagbar 设置
   F6进入，上下移动 (jk), 选中回车后会跳转。通过 help:tagbar 可以查看tagbar的说明文档 
```
" tagbar
"Bundle 'majutsushi/tagbar'

nmap <F6> :TagbarToggle<CR>
" tagbar默认去这个目录中寻找ctags，ctags的默认安装路径也是这个目录
let g:tagbar_autoclose=1 "在一个tag上按回车后，自动跳转到tag在文件的位置，并关闭tagbar
let g:tagbar_ctags_bin='/usr/bin/ctags'  " Proper Ctags locations
let g:tagbar_width=26                      " Default is 40, seems too wide
noremap <Leader>y :TagbarToggle<CR>        " Display panel with (,y)

" 启动 时自动focus
let g:tagbar_autofocus = 1

" for ruby, delete if you do not need
let g:tagbar_type_ruby = {
    \ 'kinds' : [
        \ 'm:modules',
        \ 'c:classes',
        \ 'd:describes',
        \ 'C:contexts',
        \ 'f:methods',
        \ 'F:singleton methods'
    \ ]
\ }
```
4. NERDCommenter注释工具
```
,c<Space>	切换当前行或选中内容是否注释，根据首行判断
,ci	切换当前行或选中内容是否注释，每行自己判断
,cs	有格式地注释
,cy	复制内容，然后注释
,ca	在行末添加注释符并进入insert mode
,cl	在行首添加注释符并进入insert mode
,ca	在两种注释符之间切换，如/**/和//
```
