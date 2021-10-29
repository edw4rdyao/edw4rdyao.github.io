# Ubuntu20.04使用Sublime Text3

## 下载与安装

1. 通过官网下载软件包

[Sublime Text3官网](https://www.sublimetext.com/3)

2. 在终端执行安装

```shell
$ sudo dpkg -i sublime-text_build-3211_amd64.deb
```

## 注册与激活

1. 我的Sublime版本为Build 3211, 该版本可以在Github上找到License Key

[Key Address](https://github.com/AchrafIdir/SublimeText-3211-License-Key-/blob/master/Key)
如果失效了，请自行Google

2. 输入注册码

`Sublime Text 3 -> Help ->  Enter License Key -> 粘贴注册码`

## 关闭自动更新

>为什么要关闭自动检查更新呢？因为现在Sublime Text 4已经有了，联网状态下每次打开都会提示更新，而且我们的License为Build 3版本的

1. 请执行以下步骤：

`Sublime Text 3 -> Preferences -> Settings -> 进行编辑`

2. 在大括号中加入下列元组:

```json
"update_check": false
```

## 更换主题

1. 首先安装包管理器，我们需要使用它来安装插件

`Ctrl + Shift + P` + 搜索 `install package control`

2. 安装完成后，就可以安装各种插件了

`Ctrl + Shift + P` + 搜索 `Package Control: Istall Package`

3. 等到加载插件库完成之后，搜索插件开始安装，我选择的是`Material Theme`，因为它包括图标主题，代码高亮等等

4. 安装完成之后分别设置主题和颜色主题

`Sublime Text 3 -> Preferences -> Color Theme ...`
`Sublime Text 3 -> Preferences -> Theme ...`

## 更换字体

1. 下载字体，我个人认为`Fira Code`字体美观大方，于是选择了该字体

[字体下载](https://github.com/tonsky/FiraCode/wiki/Installing)

2. 设置字体与大小粗细

`Sublime Text 3 -> Preferences -> Settings -> 进行编辑`

3. 加入以下元组进行设置

```json
"font_face": "Fira Code Medium", // 字体
"font_size": 14 // 字体大小
```