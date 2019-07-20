---
layout: post
title: 把 vim 武装成 IDE
categories: [Basic]
tags: [Tools]
description: vim 中好用的插件及其安装方式，自己动手把 vim 改造为一个轻量级的 IDE。
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
  Plugin 'VundleVim/Vundle.vim'   " required
  call vundle#end()            " required
  filetype plugin indent on    " required
  ```

* 进入  vim, 输入 :PluginInstall, 安装 Vundle。

**相关命令**

进入 vim 后，可使用以下命令。

 :PluginList        - 查看使用 Vundle 管理的全部插件  
 :PluginInstall    - 安装插件，如果末尾跟着 ! 表示更新插件  
 :PluginUpdate  - 更新插件  
 :PluginSearch foo - searches for foo; append `!` to refresh local cache  
 :PluginClean      - 确认并移除无用的插件，如想自动确认所有移除，在末尾添加 !  
:h vundle     - 查看更多详细信息  

### [NERDTree: 文件系统的树状浏览器，类似与 IDE 的 Project Explore。](https://vim8.org/scripts/script.php?script_id=1658)

安装后效果如下：

![](https://git-page.oss-cn-chengdu.aliyuncs.com/vim-plugin/nerd_tree_screenshot.png)

安装了 Vundle 后，插件安装就很方便了。

**使用 Vundle 安装步骤**

* 打开 .vimrc 文件， 在 call vundle#begin() 与 call vundle#end()  行之间，添加

  ```
  Plugin 'scrooloose/nerdtree'
  ```

* 添加完成后保存，重新进入  vim, 输入 :PluginInstall， 这时 Vundle 会自动安装 call vundle#begin()  与 call vundle#end()  命令间定义的所有插件。

安装完成后，~/.vim/bundle 包下会多出一个 nerdtree 的包，里面包含了 nerdtree 插件的全部内容。

进入 vim， 输入 :NERDTreeToggle，出现上图左侧的文件系统浏览器，表明 NERDTree 安装成功。

**设置快捷键**

当 NERDTree 使用频率比较高时，每次输入 :NERDTreeToggle 会比较麻烦，就可以为该命令设置快捷键。 

方式为打开 .vimrc, 在 call vundle#end() 行后，添加

```
map <F2> :NERDTreeToggle<CR>
```

这样我们就把 F2 映射到了 :NERDTreeToggle 命令。结尾的 \<CR\> 表示回车，可输入命令 :help key-notation 查看详细信息。保存 .vimrc 文件后，重启 vim，此时按 F2 就会出现文件系统浏览器了。

需要查看所有的快捷键设置的话，可以通过输入 :map 查看。

更详细的关于 map 配置，可以参考[这里](http://vim.wikia.com/wiki/Mapping_keys_in_Vim_-_Tutorial_(Part_1))。

### [Tagbar:显示当前文件的结构](https://www.vim.org/scripts/script.php?script_id=3465)

安装后效果如下：

![](https://git-page.oss-cn-chengdu.aliyuncs.com/vim-plugin/tagbar_screenshot.png)

图中右侧即为 tagbar 显示结果。其会对 class 对象进行归类显示，如图中是先显示类属性块，再显示类方法块。属性和方法签名前面的 +, - , #  分别表示 public, private  和 protected。

**使用 Vundle 安装步骤**

* 同样是打开 .vimrc, 在 call vundle#begin() 与 call vundle#end()  行之间，添加

  ```
  Plugin 'majutsushi/tagbar'
  ```

  Vundle 的 Plugin 管理支持多种格式，详情请参看 [Vundle README](https://github.com/VundleVim/Vundle.vim/blob/master/README.md)。上文用到的是我个人认为最便捷的 GitHub 仓库的格式，Plugin  后单引号中的值就是 GitHub 上开源仓库的唯一地址后缀。  如 tagbar  的 github 地址为 https://github.com/majutsushi/tagbar， 所以这里填入的就是 majutsushi/tagbar。NERDTree 也是使用的 GitHub 仓库管理的格式，感兴趣的小伙伴可以前去确认一下。

* 保存.vimrc 后，进入 vim, 执行 :PluginInstall。这样 tagbar 就安装好了。

安装好后进入  vim, 输入 :TagbarToggle 命令后回车，却不能出现右侧的文件结构，而是获得如下提示

```
Please download Exuberant Ctags from ctags.sourceforge.net and install it in a directory in your $PATH or set g:tagbar_ctags_bin.
```

这是由于 Tagbar 依赖于 Exuberant Ctags, 需要自行下载安装。

Mac 电脑上如果已安装了 Homebrew, 可以使用以下命令一键安装。

```
brew install ctags
```

安装好 ctags 后，再执行 :TagbarToggle 就能看到文件结构了。

**设置快捷键**

同样，我们为其设置快捷键，按官方文档建议将其映射到 F8。打开 .vimrc, 在 call vundle#end() 行后，添加

```
map <F8> :TagbarToggle<CR>
```

### 结束

以上只是我个人比较常用的 vim 插件，更多的将 vim 改装成 IDE 的插件可以参考[这里](http://vim.wikia.com/wiki/Use_Vim_like_an_IDE)。


