title: etcd v2 v3 同时存在的问题
date: 2019-04-02 17:42:00
tags: [etcd, 技术]
categories: 泛技术

------

最近使用 `etcd` 时候遇到了一个问题，发现用 `go` 代码调用 `clientv3` 写进去的 `key` 和用 `etcdctl` 的 `key` 是不一样的，而且两边可以同时读写相同的 `key`， 但是知确实不相同的。

因为这个问题查了将近两个小时，最后发现，原来 `etcd` 默认情况下是 `v2` 和 `v3` 的客户端 `API` 共存的，而两个版本的 `API` 产生和查询的数据时隔离的。

用 `etcd` 的过程中看了不少的文章，结果恰恰我看的文章里都没有提到这一点，结果当了冤大头，浪费了这么多时间。

其实现在应该大部分人都是用 `v3` 版本的 `API`，`etcdctl` 默认却是 `v2`的 `API`。不知道是出于什么考虑没有把 `v3` 版本 `API` 作为 `etcdctl` 的默认 `API` 版本。

至于我会问到这个问题，也是因为我在开发的过程中想要用 `etcdctl` 来验证我的代码写进去的数据是否正确，如果我没有用 `etcdctl` 来读 `go` 客户端写进去的数据的话，也就不会有这个问题了。如果想要让 `etcdctl` 默认用 `v3` 版的 `API`，可用在使用 `etcdctl` 之前，设置版本环境变量：

```bash
export ETCDCTL_API=3
etcdctl ...
```

如果不想每次使用 `etcdctl` 是都设置，可以在 `.bashrc` 或者 `.bash_profile` 加入该语句：

```bash
export ETCDCTL_API=3
```

