## brew 使用

### 概念

Homebrew, Mac 的包管理工具，类似 yum 和 apt-get


### 基础知识

Homebrew 的两个术语：

- Formulae：软件包，包括了这个软件的依赖、源码位置及编译方法等；
- Casks：已经编译好的应用包，如图形界面程序等

Homebrw相关的几个文件夹用途：

- bin：用于存放所安装程序的启动链接（相当于快捷方式）
- etc：brew安装程序的配置文件默认存放路径
- Library：Homebrew 系统自身文件夹
- Cellar：通过brew安装的程序将以 [程序名/版本号] 存放于本目录下

### 常用命令

``` shell
# 查看brew版本
$ brew -v

# 查看帮助
$ brew --help
$ brew -h

# brew 更新
$ brew update

# 查看包的详细信息
$ brew info xxx

# 列出本地软件库
$ brew list

# 查看软件库版本
$ brew list --versions

# 搜索软件包（支持正则表达式）
$ brew search xxx

# 卸载软件包
$ brew uninstall xxx

# 安装包
$ brew install xxx
$ brew install [--cask] xxx
```

> cask 加与不加的区别：
>
> brew 是下载源码解压，然后 ./configure && make install ，同时会包含相关依存库，并自动配置好各种环境变量
>
> brew cask 是针对已经编译好了的应用包（.dmg/.pkg）下载解压，然后放在统一的目录中（Caskroom），省掉了自己下载、解压、安装等步骤

> - brew install 用来安装一些不带界面的命令行工具和第三方库
> - brew cask install 用来安装一些带界面的应用软件