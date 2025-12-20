# 使用Git实现Windows与Mac文件同步

2024/09/10 

今天晚上成功在windows上安装了Git，并且和GitHub的Computer-Organization仓库链接上了，以后就能很方便地同步win和mac的文件了。(再也不用处处登录微信啦)

记一下命令，以免遗忘。

感叹一下，Git 真好哇:)

---

## 操作流程

### 首次

通常先在 GitHub 上创建好仓库，再 `git clone` 到本地

克隆 GitHub 仓库 (最好在终端中操作，Git Bash 好像不能复制 @github...):

```bash
$ git clone git@github.com:tsai-shiyuan/balabala
```

会自动在本地 `git init`

在本地编辑好文件后，推送到本地 Git 及 GitHub 上:

```bash
$ git add .
$ git commit -m "description"
$ git push -u origin main
```

### 后续

如果 `git clone` 后再次 `git clone` 会报错:

```bash
fatal: destination path 'Computer-Organization' already exists and is not an empty directory.
```

according to [Rafa Alonso](https://stackoverflow.com/users/7335975/rafa-alonso):

For those coming here wishing to  clone a repository that already has contents in an existing folder that already contains files/folders, follow the following steps:

```bash
cd /	# point to your desired folder
git init
git remote add origin <your-repo-url>
git pull
git checkout main -f
git branch --set-upstream-to origin/main
```

- `cd` 到已存在的文件夹内，即 `Computer-Organization` 内，和之前 `cd` 到上一级不同
- 自己输入 `git remote add origin <url>` 会报错，`error: remote origin already exists` ，因为之前已经 `clone` 过？所以不加这句就好
- 执行最后两句前文件好像就成功拷贝了？所以不用加
