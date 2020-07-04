title: 一个让go的exec支持管道的小窍门
date: 2019-05-16 17:25:03
tags: [go, 技术, bash]
categories: go

------

最近在一个 `golang` 开发的项目中需要用 `exec` 库调用外部 `shell` 命令，这个命令中用到了管道(`pipline`)。

由于是第一次用 `exec` 库，我想当然的把代码写成了这个样子：

```go
package main

import "fmt"
import "os/exec"

func main() {
    cmdStr := fmt.Sprintf("ls -l %s | head -n %d", ".", 10)
    cmd := exec.Command(cmdStr)
    if out, err := cmd.CombinedOutput(); err != nil {
        fmt.Errorf("Error: %v\n", err)
    } else {
        fmt.Printf("Success: %s\n%s\n", cmdStr, out)
    }
}
```

运行的时候发现这段代码是不能正确的工作的，经过在网上一通搜索后，发现网上给出的方案大致都是这个样子的：[https://stackoverflow.com/questions/10781516/how-to-pipe-several-commands-in-go](https://stackoverflow.com/questions/10781516/how-to-pipe-several-commands-in-go)。这种方案确实是解决了问题，但是看起来却十分的复杂，我只是想调用一句 `shell` 命令而已，却要多写上十几行的 `go` 代码。

有没有更简单的方法呢？既然 `go` 自身处理不了，是否可以用 `go` 之外的工具来解决呢？

<!-- more -->

然后就有了这个曲线救国的方案，`exec` 不再直接执行这个带有管道的命令，而是执行 `sh` 命令，然后我们的带有管道的 `shell` 命令作为参数传给 `sh` 来运行，类似于：

```sh
sh -c "ls -l . | head -n 10"
```

至此，问题迎刃而解，比网上找到的方案简单了很多。落实到 `go` 代码大概就是下面的样子：

```go
package main

import "fmt"
import "os/exec"

func main() {
    cmdStr := fmt.Sprintf("ls -l %s | head -n %d", ".", 10)
    cmd := exec.Command("sh", "-c", cmdStr)
    if out, err := cmd.CombinedOutput(); err != nil {
        fmt.Errorf("Error: %v\n", err)
    } else {
        fmt.Printf("Success: %s\n%s\n", cmdStr, out)
    }
}
```

其实原则上，不局限于本题中管道的情形，这个方案还可以用在更多的需要调用复杂的 `shell` 命令的场合。

