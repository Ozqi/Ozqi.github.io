
# **GIT**
![img](./image/Ubuntu.assets/1352126739_7909-16405943342913.jpg)

**工作区**：就是你在电脑里能看到的目录。

**暂存区**： stage 或 index。一般存放在 **.git** 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。

**版本库**：工作区有一个隐藏目录 .git，这个是 Git 的版本库。


![img](./image/Ubuntu.assets/git-command-16405943413395.jpg)
## 拉取代码（开始工作）

```bash
git fetch # remote -> loacal repo
```
更新了远程分支的信息之后，可以**基于某远程分支，新建本地分支。**
```bash
git checkout -b new-local-branch origin/remote-branch-name
```

也用下面命令把这些旧的远程跟踪分支干掉：

```bash
git fetch --prune github
```


## 丢弃更改
 1. 丢弃工作目录中所有未暂存（unstaged）的更改，即还没有通过 `git add` 暂存起来的更改，可以使用以下命令：

```bash
git checkout -- .
```
2. 已经add，但未commit的更改：
先取消这些文件的暂存状态，然后才能丢弃更改：
```bash
git reset HEAD -- .
git checkout -- .
```
3. 重置到最近一次commit：
```bash
git reset --hard HEAD
#! 使用 `--hard` 选项会丢失所有未提交的更改，请确保这是你真正想要做的，因为这个操作是不可逆的。 
```
4. 已经提交过,想重置为远程仓库
如果你的本地分支落后于远程仓库，且你想放弃所有本地提交，使本地分支与远程同步，可以首先获取最新的远程状态，然后进行硬重置：
```bash
git fetch origin
git reset --hard origin/your-branch-name
```


## 推代码（提交PR）
1. 更新，因为在此期间别人可能更新了代码
```bash
git checkout ocf-for-AICache  # 切换到主分支
# git fetch 
git pull origin ocf-for-AICache # 拉取最新的代码

git checkout your-feature-branch  # 切换到你的功能分支
git merge ocf-for-AICache  # 合并主分支的最新代码

# ...
# 解决冲git checkout ocf-for-AICache  # 切换到主分支
# git fetch 
git pull origin ocf-for-AICache # 拉取最新的代码

git checkout your-feature-branch  # 切换到你的功能分支
git merge ocf-for-AICache  # 合并主分支的最新代码

# ...
# 解决冲突
# ...
```
2. 推送
```bash
# 解决冲突后，添加文件并提交。
git add .
git commit -m "Resolved merge conflicts"

# 可选（合并commit）

git push origin your-feature-branch
# 推送到自己的分支（最好与本地同名），然后会弹出MR链接

# 推送到指定分支：
git push origin your-feature-branch:remote-branch-name

```
3. 新建PR（一般会弹出新建PR的链接）
4. 等待被合并，一般可以选择“合并后删除远程分支”
## 合并多个commit

```bash
# 合并最近n次commit
git rebase -i HEAD~n
# 然后会进入交互，一般如果除了第一行的第一个词都改成f （fixup），合并后会以最新的message为合并后的message

```



## 网络问题
代理&&测试：
先为 git 配置 HTTP/HTTPS 代理指向本机 7890，然后用 git 的网络调试输出检测连通性，然后再尝试 push。
```bash
git config --global http.proxy http://127.0.0.1:7890 && git config --global https.proxy http://127.0.0.1:7890 && GIT_CURL_VERBOSE=1 git ls-remote https://github.com/LiuziqiOvO/clash-easy-cli.git | head -n 1 | cat
```


配置 GITHUB SSH 密钥
1. 生成 SSH 密钥（如果未配置）：
```bash
ssh-keygen -t rsa -b 4096
#直接按回车，使用默认路径，设置一个密码。
```

2. 将公钥添加到 GitHub：
```bash
   cat ~/.ssh/id_rsa.pub
```
  将输出的内容复制到 GitHub → **Settings** → **SSH and GPG keys** → **New SSH Key**。

3. 测试 SSH 连接：
   ```
   ssh -T git@github.com
   ```
   成功时输出：
   ```
   Hi LiuziqiOvO! You've successfully authenticated, but GitHub does not provide shell access.
   ```

4. 切换为 SSH 连接：
查看当前 Git 远程 URL：
```
git remote -v
```
示例输出（如果是 HTTPS）：
```
origin  https://github.com/LiuziqiOvO/SPDK.ocf.git (fetch)
origin  https://github.com/LiuziqiOvO/SPDK.ocf.git (push)
```
将远程 URL 从 `HTTPS` 改为 `SSH`：
```
git remote set-url origin git@github.com:LiuziqiOvO/SPDK.ocf.git
```
### Github token登录

现在Github 登录密码强制使用 token 了
​```bash
git remote set-url origin https://token@github.com/username/project.git

### push 操作经常超时

```bash
git config --global http.proxy http://127.0.0.1:1080

git config --global https.proxy http://127.0.0.1:1080
```

```bash
#取消全局代理 //不是很懂
git config --global --unset http.proxy

git config --global --unset http.proxy
```

也可以不使用http，在 git remote里把仓库地址改成git开头。


> 如何只下载某个仓库的一部分文件

工具：http://tool.mkblog.cn/downgit/#/home


## 大文件问题

下载安装 git lfs https://git-lfs.com/

https://blog.csdn.net/wzk4869/article/details/131661472

```bash
git lfs install
#追踪大文件
git lfs track "*.pptx"
#重新add commit
```


- GitHub 拒绝推送，因为检测到超过 100MB 的大文件
- 即使设置了 `.gitignore`，已追踪的文件仍会被推送

## 取消追踪，gitignore

1. 从 Git 追踪中移除大文件并重新提交：
```bash
git rm --cached -r <目录>
git commit -m "chore: remove directory from git tracking"
```

2. 清理历史记录中的大文件：
```bash
git filter-branch --force --index-filter  --prune-empty --tag-name-filter cat -- --all
```

3. 清理引用日志：
```bash
git for-each-ref --format='delete %(refname)' refs/original/ | git update-ref --stdin
```

4. 清理 Git 垃圾：
```bash
git reflog expire --expire=now --all 
git gc --prune=now --aggressive
```

5. 强制推送清理后的仓库：
```bash
git push -f origin main
```

- 大文件最好使用 Git LFS 管理
- `.gitignore` 只对未追踪的文件有效
- 已追踪的文件需要先移除追踪，再设置忽略规则

## git卡死(移除锁)
危险
```bash
cd /home/liuziqi/lzq/Proj/spdk && rm -f .git/index.lock && git add . && git commit -m 'update: 更新了测试脚本在 lzq_docker文件夹' && git push
```

## submodule
有些时候要手动sync一下，


|     |                                           |                      |
| --- | ----------------------------------------- | -------------------- |
| 1   | 编辑 `.gitmodules` 文件                       | 将 URL 从 HTTP 改成 SSH  |
| 2   | `git submodule sync`                      | 把新的 URL 同步到 Git 内部配置 |
| 3   | `git submodule update --init --recursive` | 更新子模块内容              |
|     |                                           |                      |

### GITHUB访问问题

gitpush的时候让输入账号密码
已经不支持账号密码认证了。
密码的地方要填 Personal Access Token
https://github.com/settings/tokens/new

或者配置SSH也是可以的。
