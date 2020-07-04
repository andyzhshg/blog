title: 用 AppleScript 一键自动处理选中的文本
date: 2019-07-24 18:31:50
tags: [技术, mac, osx, AppleScript]
categories: 技术

------

这篇文章提到的方法是为了解决我在工作中遇到的一个实际问题，问题大概是这个样子的：

我的一个服务程序使用 `golang` 写的，其中的日志用了[`logrus`](https://github.com/sirupsen/logrus) 这个库，这个库本身很好用，但是我碰到一个问题 ，就是我的 log 中有一些数据本身就是以 json 格式输出的，但是 logrus 在日志文件中会默认的给输出中的双引号转义，最后看到的日志大概是这个样子的：

```
time="2019-07-24T10:39:06Z" level=info msg="[Out] {\"id\":257944,\"type\":1,\"timestamp\":1563964746233023125,\"offer_id\":741807,\"status\":{\"code\":0,\"message\":\"ok\"},\"data\":{\"symbol\":4,\"topic\":\"rest2engine_12_13\"}}"
time="2019-07-24T10:39:06Z" level=info msg="[Out] {\"id\":257945,\"type\":3,\"timestamp\":1563964746233044370,\"trigger\":{\"id\":741807,\"is_bid\":false,\"amount\":\"4380000000\",\"clear\":true,\"balance\":\"0\",\"data\":{\"symbol\":4,\"topic\":\"rest2engine_12_13\"}},\"match\":[{\"id\":740840,\"price\":\"2074400000000\",\"amount\":\"1940000000\",\"clear\":true,\"balance\":\"0\",\"data\":{\"symbol\":4,\"topic\":\"rest2engine_12_13\"}},{\"id\":741724,\"price\":\"2074400000000\",\"amount\":\"2440000000\",\"clear\":false,\"balance\":\"10735020000000000000000\",\"data\":{\"symbol\":4,\"topic\":\"rest2engine_12_13\"}}],\"status\":{\"code\":0,\"message\":\"ok\"}}"
```

一般情况下，需要人肉去读这些日志的时候我会把 json 数据拷贝出来粘贴到一个可以格式化 json 的编辑器中。但是因为这个带了转义符号的数据已经不是合法的 json 了，所以 json 编辑器是无法直接处理的。

我已开始的做法是把数据拷贝到文本编辑器中，然后通过全局替换的方式把转义字符替换掉，即用 `"` 替换 `\"` 。这种方式其实在我这个场景工作的很好，只是比较麻烦，每次都要通过文本编辑器全局替换一次。

那么有没有更加有效率的方式呢？一开始我是想到可以自己写一个小程序专门的来处理这个替换过程，不过每次都需要调用小程序，比贴到文本编辑器全局替换也省不了时间。然后我就突然想到了是不是可以用 mac 的自动操作 Apple Script 来完成这件事，能想到这个是因为我之前在一篇文章里学到过通过 Apple Script 添加 VS Code 的右键菜单。

<!--more-->

其实所谓的 Apple Script 基本上是不需要自己动手写代码的，只需要在图形化的编辑器里拖一拖选项就大致可以完成工作了。我的最终的成果就是可以在终端选中一段数据，然后通过右键选择我编写好的脚本动作，就把选中的文字做全局替换，然后将 json格式化，并且拷贝进剪贴板，查看的时候只需要在文本编辑器 cmd+v 将剪贴板的内容粘贴即可。比起之前的流程，简直是舒服了太多。

以下是 Apple Script 的制作过程：

1. 打开 `自动操作` 程序

   ![](http://up4dev.oss-cn-qingdao.aliyuncs.com/text-process-by-apple-script/tpbas-1.png)

2. 选 `新建文稿`，文稿类型为 `快速操作`

   ![](http://up4dev.oss-cn-qingdao.aliyuncs.com/text-process-by-apple-script/tpbas-2.png)

3. 在搜索栏搜索 `shell` ，选择 `运行 shell 脚本` 组件，拖入工作区，并填入 `sed 's/\\\"/\"/g' | /usr/local/bin/jq` 作为脚本内容。

   ![](http://up4dev.oss-cn-qingdao.aliyuncs.com/text-process-by-apple-script/tpbas-3.png)

4. 在搜索栏搜索 `剪贴板`，选择 `拷贝至剪贴板` 组件，拖入工作区。

   ![](http://up4dev.oss-cn-qingdao.aliyuncs.com/text-process-by-apple-script/tpbas-4.png)

5. 保存脚本，比如我命名为 `Logrus Trans`

6. 至此脚本就编写完成了，使用的效果大概是这样的

   ![](http://up4dev.oss-cn-qingdao.aliyuncs.com/text-process-by-apple-script/tpbas-5.png)



可能需要解释以下的是步骤 3 中的这段脚本： `sed 's/\\\"/\"/g' | /usr/local/bin/jq` 

这是一个用管道串联起来的字符流处理，实际上 `sed` 之前是 AppleScript 传递过来的字符流，也就是我们选中的文本，`sed 's/\\\"/\"/g'` 会将文本中的 `"` 替换成 `\"` ，`/usr/local/bin/jq ` 则将传递过来的文本中的 json 进行格式化。

步骤 4 会将步骤 3 的输出的文本拷贝到剪贴板中，这样在使用的时候直接用 `cmd + v` 粘贴到需要的地方就可以了。

这个例子是一个特例，其他人大概不会用到一模一样的情境，不过这里提供了一种思路，可以利用 AppleScript 快捷的处理繁复的工作。