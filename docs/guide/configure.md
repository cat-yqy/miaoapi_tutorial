# 在 Codex 中配置喵API

在 [MiaoAPI](https://miaoapi.site) 创建 API Key 后，从下面三种方式中任选一种。三种方式最终效果一致，推荐直接修改配置文件（方法一）。

!!! tip "先创建密钥"
    登录 MiaoAPI 后进入密钥管理，创建一个 `codex` 分组的密钥。如果已购买套餐，请选择套餐对应的专属分组。

## 方法一：修改配置文件（推荐）

首先进入你的账号，然后创建API key。

![操作截图](../assets/miaoapi/img-11.webp)

这里点击“创建密钥”

![操作截图](../assets/miaoapi/img-12.webp)

这里选择codex分组即可（如您订阅了套餐，请新建密钥选择套餐的专属分组）

![操作截图](../assets/miaoapi/img-13.webp)

![操作截图](../assets/miaoapi/img-14.webp)

这里点击使用密钥

![操作截图](../assets/miaoapi/img-38.webp)

复制弹出的界面的两个配置，记住相对应的路径（如果没有该名字文件，则创建文件），然后把内容直接全部复制进去，保存即可。

![API Key 已打码的配置截图](../assets/miaoapi/img-39.webp)

下面演示一下mac电脑如何找到这个文件夹和目录

### macOS / Linux 配置

首先打开访达，找到这个小房子图标，也就是Home目录，点进去。

![操作截图](../assets/miaoapi/img-40.webp)

这个时候你还看不到名字叫.codex的文件夹，所以需要按command+shift+.（也就是“.”这个键！！），这样你就能看到隐藏文件夹.codex了，咱们点进去。

如果你这一步之后还是没在隐藏文件夹找到.codex文件夹，说明你codex还没初始化，请先打开一次codex或者打开一次cli创建文件夹。

![操作截图](../assets/miaoapi/img-41.webp)

可以看到这两个我们需要的文件了（如果没有请自己创建这两个同名文件）。

![操作截图](../assets/miaoapi/img-42.webp)

把我们的内容复制进去配置完成后直接在终端

```bash
codex
```

或者直接打开codex app即可使用codex cli或者codex app了，记得重启一下。

后续可以在Codex cli（输入指令/model）或者codex app（直接在对话框下面选择切换）切换模型为gpt-5.5，现在默认配置是选择gpt-5.4。

![操作截图](../assets/miaoapi/img-43.webp)

若您是winodws或者linux系统，配置方法也类似，如您还存在其他问题，可以优先询问一下Deepseek，也欢迎在群内咨询～

### Windows 配置

对于windows系统，可以直接按Win键+R键，然后输入%userprofile%\\.codex并直接回车，可以直接打开.codex目录。

![操作截图](../assets/miaoapi/img-44.webp)

感谢您对喵API的支持！

## 方法二：CC Switch

<https://github.com/farion1231/cc-switch>

CC Switch 也可以切换不同中转站或官方订阅，保持本地长久会话不被破坏。

首先安装CC switch

![操作截图](../assets/miaoapi/img-20.webp)

找到releases，点击这里弹出来的最新版本

![操作截图](../assets/miaoapi/img-21.webp)

下滑找到相对应的版本

![操作截图](../assets/miaoapi/img-22.webp)

这里绿框是linux，蓝框是macos，红框是windows，大家根据自己的电脑系统下载安装即可。

![操作截图](../assets/miaoapi/img-23.webp)

安装完后打开一下，你就可以进入CC swtich啦～（这里dz已经配置好了）

![操作截图](../assets/miaoapi/img-24.webp)

下面讲一下如何导入我们的某个密钥配置

首先进入你的账号，然后创建API key。

![操作截图](../assets/miaoapi/img-11.webp)

这里点击“创建密钥”

![操作截图](../assets/miaoapi/img-12.webp)

这里选择codex分组即可（如您之后订阅了套餐，请新建密钥选择套餐的专属分组）

![操作截图](../assets/miaoapi/img-13.webp)

![操作截图](../assets/miaoapi/img-14.webp)

接着点击“导入到CCS”这个按钮。

![操作截图](../assets/miaoapi/img-25.webp)

允许即可！

![操作截图](../assets/miaoapi/img-26.webp)

这里点“导入”确定。

![操作截图](../assets/miaoapi/img-27.webp)

OK，我们现在有两个配置了。

![操作截图](../assets/miaoapi/img-28.webp)

可以改一下名，比如我把刚导入的改为test，然后点击启动，就可以自动切换密钥啦！（这里的查询失败是bug，不影响）

![操作截图](../assets/miaoapi/img-29.webp)

![操作截图](../assets/miaoapi/img-30.webp)

如果您还有官方订阅（或其他中转站），可以点击右上角的➕号

![操作截图](../assets/miaoapi/img-31.webp)

选择Openai Official

![操作截图](../assets/miaoapi/img-32.webp)

点击添加

![操作截图](../assets/miaoapi/img-33.webp)

![操作截图](../assets/miaoapi/img-34.webp)

再点击启用官方订阅。

![操作截图](../assets/miaoapi/img-35.webp)

随后打开中转，随便找个文件夹，输入codex logout。

![操作截图](../assets/miaoapi/img-36.webp)

出现successfully logged out之后，再输入codex，利用网页认证登录您的gpt账号即可，会话将不会消失。

![操作截图](../assets/miaoapi/img-37.webp)

如您需要切换回中转站，可以点击中转站的启用即可。

## 方法三：Cockpit Tools

<https://github.com/jlcodes99/cockpit-tools>

Cockpit Tools 可以方便地切换不同中转站或官方订阅，保持本地长久会话不被破坏。

首先安装Cockpit tools。

![操作截图](../assets/miaoapi/img-01.webp)

找到releases，这里点击弹出来的最新版本

![操作截图](../assets/miaoapi/img-02.webp)

这里红色是macos版本，黄色是linux，蓝色是windows版本。

![操作截图](../assets/miaoapi/img-03.webp)

下载完成后打开，这里以macos为例。

![操作截图](../assets/miaoapi/img-04.webp)

点击左边这个Openai标识。

![操作截图](../assets/miaoapi/img-05.webp)

点击添加账号

![操作截图](../assets/miaoapi/img-06.webp)

点击api key。

![操作截图](../assets/miaoapi/img-07.webp)

选择自定义。

![操作截图](../assets/miaoapi/img-08.webp)

下拉，红色框填API key（需要去网页密钥管理创建），绿色框填https://miaoapi.site/，黄色框随便填，比如miaoapi。

![操作截图](../assets/miaoapi/img-09.webp)

![操作截图](../assets/miaoapi/img-10.webp)

下面讲一下怎么创建API key。

首先进入你的账号，然后创建API key。

![操作截图](../assets/miaoapi/img-11.webp)

这里点击“创建密钥”

![操作截图](../assets/miaoapi/img-12.webp)

这里选择codex分组即可（如您之后订阅了套餐，请新建密钥选择套餐的专属分组）

![操作截图](../assets/miaoapi/img-13.webp)

![操作截图](../assets/miaoapi/img-14.webp)

点击红色框的复制可以复制密钥，点击蓝色框的复制可以复制api端点，也即站点。

![操作截图](../assets/miaoapi/img-15.webp)

下面讲一下怎么切换官方订阅。

还是一样的，点击添加账号。

![操作截图](../assets/miaoapi/img-16.webp)

点击红色框oauth授权，再点击从浏览器打开（或者复制之后到浏览器打开）

![操作截图](../assets/miaoapi/img-17.webp)

![操作截图](../assets/miaoapi/img-18.webp)

一直登录即可，之后授权完会出现“返回失败”界面，此时需要手动复制浏览器顶部“返回链接”（http://localhost开头的一串链接）到此框里即可，然后点我已授权，继续。

![操作截图](../assets/miaoapi/img-19.webp)
