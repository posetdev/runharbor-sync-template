# RunHarbor 同步仓库

这个仓库负责把你在 Gitee 上的代码镜像到本组织的同名 GitHub 仓库。镜像推送后，目标仓库自带的 GitHub Actions 会照常触发，并跑在你自己 AWS 里的 RunHarbor runner 上。

每个组织只需要一个同步仓库，名字必须是 `runharbor-sync`。请按 RunHarbor 控制台「代码同步」页的向导操作，不需要手动修改这个仓库。

## 它是怎么工作的

1. 你的 Gitee 仓库收到推送后，通过 WebHook 通知 RunHarbor。
2. RunHarbor 让你 AWS 里的 Connector 运行本仓库的 `RunHarbor 同步` workflow，并只告诉它要同步哪个目标仓库。
3. 这个 workflow 在你自己的 RunHarbor runner 上运行：从 Gitee 拉取全部分支和标签，再用目标仓库的 deploy key 推到 GitHub。

## 谁能决定从哪拉、推到哪

每个目标仓库对应本仓库的一个 Actions secret，名字是 `RUNHARBOR_SYNC_<仓库名>`，内容是你在控制台里生成、再亲手粘贴进来的「同步凭据」。凭据里有三样东西：

- 源地址；
- 目标仓库名；
- 一把只属于这一个目标仓库的 SSH 私钥。

对应的公钥由你加到目标仓库的 Deploy keys 里，并勾选写权限。私有的 Gitee 仓库还要把它加到 Gitee 的「部署公钥」里。

RunHarbor 的服务端只能告诉 workflow「同步哪个目标」，既看不到凭据，也改不了源地址。凭据里写明了它属于哪个目标，用错了 secret，workflow 会直接拒绝运行。

## 需要注意

- 目标仓库是纯镜像。源仓库的分支和标签会原样覆盖过去；只在 GitHub 上存在的分支和标签会被删除。GitHub 不允许删除默认分支，遇到这种情况会给出提醒并保留该分支。
- 不支持 Git LFS。GitHub 拒收单个超过 100 MB 的文件。
- 同步任务跑在 `rh-x64-2vcpu-4gib` 规格的 RunHarbor runner 上。
