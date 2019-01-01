---
layout: post
title: 把 vim 武装成IDE -- 好用的插件及其安装方式 
categories: [Basic]
tags: [工具]
description: 
---
### [Vundle: vim 的插件管理器](https://github.com/VundleVim/Vundle.vim)

安装方式：

* 将 Vundle 拷贝到 ~/.vim/bundle/Vundle.vim

  ```
  git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
  ```

*  在根目录下新建 .vimrc 文件，添加如下内容。该文件中 “ 开始表示注释。

* ```
  set nocompatible              " be iMproved, required
  filetype off                  " required
  
  " set the runtime path to include Vundle and initialize
  set rtp+=~/.vim/bundle/Vundle.vim
  call vundle#begin()
  " " alternatively, pass a path where Vundle should install plugins
  " "call vundle#begin('~/some/path/here')
  "
  " let Vundle manage Vundle, required
  Plugin 'VundleVim/Vundle.vim'
  " All of your Plugins must be added before the following line
  call vundle#end()            " required
  filetype plugin indent on    " required
  " To ignore plugin indent changes, instead use:
  " filetype plugin on
  "
  " Brief help
  " :PluginList       - lists configured plugins
  " :PluginInstall    - installs plugins; append `!` to update or just
  " :PluginUpdate
  " :PluginSearch foo - searches for foo; append `!` to refresh local cache
  " :PluginClean      - confirms removal of unused plugins; append `!` to
  " auto-approve removal
  "
  " see :h vundle for more details or wiki for FAQ
  " Put your non-Plugin stuff after this line
  ```

* 进入  vim, 输入 :PluginInstall, 安装 Vundle。

### [NERDTree: 文件系统的树状浏览器，类似与 IDE 的Project Explore。](https://vim8.org/scripts/script.php?script_id=1658)

安装后效果如下：

![](/assets/image/vim-plugin/nerd_tree_screenshot.png)

安装了 Vundle 后，插件安装就很方便了。

##### 使用 Vundle 安装步骤

* 打开 .vimrc 文件， 在 call vundle#begin() 与 call vundle#end()  行之间，添加

  ```
  Plugin 'scrooloose/nerdtree'
  ```

* 添加完成后保存，重新进入  vim, 输入 :PluginInstall， 这时 Vundle 会自动安装 call vundle#begin()  与 call vundle#end()  命令间定义的所有插件。

安装完成后，~/.vim/bundle 包下会多出一个 nerdtree 的包，里面包含了 nerdtree 插件的全部内容。

进入 vim， 输入 :NERDTreeToggle，出现上图左侧的文件系统浏览器，表明 NERDTree 安装成功。

##### 设置快捷键

当 NERDTree 使用频率比较高时，每次输入 :NERDTreeToggle 会比较麻烦，就可以为该命令设置快捷键。 

方式为打开 .vimrc, 在 call vundle#end() 行后，添加

```
map <F2> :NERDTreeToggle<CR>
```

这样我们就把 F2 映射到了 :NERDTreeToggle 命令。保存 .vimrc 文件后，重启 vim，此时按 F2 就会出现文件系统浏览器了。




