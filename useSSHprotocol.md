# 1. 为什么使用 SSH protocol
在[主系列教程]()中, 我们使用 **HTTPS 协议** 连接远程仓库.  
**HTTPS 协议** 是目前比较受用户青睐的连接方式, 用以下命令可以查看到当前 **origin** 指向的是以 `https` 开头的 URL:
```git
git remote -v
```
HTTPS 与 SSH 都是一种安全的网络连接协议:
- 使用 HTTPS 协议对远程仓库操作需要提供用户的**账号和密码** (没有提示输入是因为 git 保存了你的账号密码);
- 使用 SSH 协议对远程仓库操作, 首先需要匹配保存在电脑中的**私钥**, 其次如果用户对私钥设置了密码, 还需提供该**密码**.

对于公开的项目, 用何种协议或许无关紧要;  
但对于私有的 (或团队管理) 的项目, 我们应该选择一种更加安全的信息保护措施.  
相较 HTTP, HTTPS 已经足够安全; 但由于 SSH 基于 **非对称加密技术**, 它的安全性又更胜一筹.

# 2. 删除 SSH keys
在某些情况下, 如果绑定的密钥出现问题, 我们需要需要删除它.  
后续节中, 如果发现密钥出问题, 请回到这步删除你的密钥.  
1. 进入储存了密钥的文件夹 (默认在 ***C:/Users/你的用户名/.ssh/*** );
2. 删除出现问题的密钥对(一个**没有后缀名的文件**(private key) 和 一个**同名但后缀名为.pub的文件**(public key));
3. 删除 **known_hosts** 和 **known_hosts.old** (如果有);
4. 如果有 **config** 文件, 用记事本打开, 删除其中出现问题的密钥配置;
5. 登录 github 账号, 在 **Settings** 页面左侧栏中找到 **SSH and GPG keys**, 删除已有的 ssh 密钥.

# 3. 创建 SSH keys
在 git 中存在两种算法生成密钥, 一种是 `rsa`, 另一种是 `ed25519`.  
`ed25519` 是一种 ECC 算法, 比起传统的 `rsa` 更加现代化和高效.  
**因此推荐使用 `ed25519` 算法生成密钥.**  
1. 在任意位置右键打开 git bash, 在终端输入以下命令 (建议将引号内内容替换为自己的邮箱):
     ```git
     ssh-keygen -t ed25519 -C "your@email.address" -f ~/.ssh/my_github_ed25519
     ```
   - `ssh-keygen` 表示生成 `ssh` 密钥;
   - `-t ed25519` 表示使用 `ed25519` 算法; 如果使用 `rsa` 算法, 建议输入 `-t rsa -b 4096`, 即生成 4096 bits 的密钥;
   - `-C "comments"` 是对该密钥的**说明**, 引号内可以填写邮箱或者**任何文字**;
   - `-f ~/.ssh/my_github_ed25519` 指出了密钥的生成**路径**以及密钥的**文件名**, 文件名可以依据自己的需求更改, 密钥可在 *C:/Users/你的用户名/.ssh/* 文件夹内找到.
2. 接着终端提示**设置密码**, 当远程的公钥与电脑的私钥匹配后, 用户希望进一步操作则需要输入密码.  
   **强烈建议设置密码, 但请确保自己记得住该密码.**  
   在你输入密码时, 终端的界面上不会显示出白色的密码字符, 这是对周围环境的防范.
3. **再次输入与刚刚相同的密码**, 匹配成功后显示密钥成功生成, 并输出了密钥的指纹和随机图像. 可以忽略这些内容;
4. 进入储存了密钥的文件夹 (默认在 ***C:/Users/你的用户名/.ssh/*** ), 右键**新建文本文档, 取名为 "config" (不要保留.txt后缀名)**;  
   右键选择用记事本打开, 在里面输入以下内容 (**最后一行是密钥路径和文件名, 注意根据自己的情况更改**):
   ```
   Host github.com
   Hostname github.com
   # ProxyCommand connect -S 127.0.0.1:7890 %h %p
   User git
   PreferredAuthentications publickey
   IdentityFile ~/.ssh/my_github_ed25519
   ```
   - **关于 Host 和 Hostname**  
     Host 是别名, Hostname 是域名;  
     例如命令 `ssh -T git@github.com` 中 `github.com` 是 Hostname;  
     如果将 Host 设置为 `github`, 那么只需输入 `ssh -T github` 就行;  
     但此时输入 `ssh -T git@github.com` 会出错,  
     因此**务必统一将 Host & Hostname 设置为 `github.com` 以避免莫名其妙的错误.**
   - **关于 Proxy**  
     上文给出的配置命令中用 `#` 注释掉了 `ProxyCommand`, 如果希望配置代理, 请删除 `#` 并按照以下规则改动命令和端口:  
     SOCKS代理: `ProxyCommand connect -S localhost:1080  %h %p`  
     HTTP代理: `ProxyCommand connect -H localhost:1080  %h %p`  
     **注:** 本文是基于windows平台撰写的, 上述代理方式使用了 Git for Windows 同捆的 connect.exe. 如果是 Linux 平台, 需要额外安装 connect-proxy.
   - **关于 127.0.0.1 和 7890**
     127.0.0.1 指本地ip地址, 7890 指代理的端口.

# 4. 连接到 Remote Repository
1. 进入储存了密钥的文件夹 (默认在 ***C:/Users/你的用户名/.ssh/*** ),  
   用记事本打开刚刚创建的密钥对的**公钥 (你取的文件名.pub)**,  
   复制里面的**所有内容**.
2. 登录 github 账号,  
   在 **Settings** 页面左侧栏中找到 **SSH and GPG keys**,  
   点击右侧按钮 **New SSH key**,  
   随便取个 Title,  
   在 Key 的输入框中**粘贴刚刚复制的公钥**.  
3. 在任意位置右键打开 git bash, 在终端输入以下命令:
   ```git
   ssh -T git@github.com
   ```
   终端提示输入密码, 如果是新的密钥, 成功后会提示该没要还没被授权, 提问是否授权;  
   输入 `yes`, 成功后会有类似于: `Hi jamesnulliu! You've successfully authenticated, but GitHub does not provide shell access.` 的提示;  
   如果终端提示: `git@github.com: Permission denied (publickey).`,  那请检查 **第3节第4步** 中的 **config 文件** 是否配置正确, 如果还是不行就回到 **第2节** 删除 ssh key 重新来过.
4. 再次输入以下命令:
   ```git
   ssh -T git@github.com
   ```
   输入密码后终端给出类似于 `Hi jamesnulliu! You've successfully authenticated, but GitHub does not provide shell access.` 的提示, 说明密钥连接成功.

# 5. 用 SSH protocol 进行项目管理
## 5.1. New Repository
和 HTTPS 协议唯一不同的地方在于, 在 github 上复制项目链接的时候选择 HTTPS 旁边的 SSH, 点小方块复制连接.  

![image](/image/ssh-01.jpg)

在外部文件夹使用以下命令 clone 储存库 (更换为你的储存库链接):
```git
git clone git@github.com:jamesnulliu/test.git
```
clone 完成后将工作区切换到 clone 下来的文件夹 (本地储存库) 内, 在终端输入以下命令:
```git
git remote -v
```
可以看到现在 origin 指向的已经是 ssh 协议的链接了.

## 5.2. 更改已有 Local Repository 的连接方式
用 `git remote -v` 查看本地储存库链接方式;  
如果本地储存库已经用 https 协议链接, 请遵输入以下命令更改 origin 指向的 URL (**xxx 替换成 ssh 连接地址**).  
```
git remote set-url origin xxxxxxx
```

---

**参考:**  
[git@github.com: Permission denied (publickey)](https://stackoverflow.com/questions/57734669/gitgithub-com-permission-denied-publickey)  
[Git SSH密钥删除与创建](https://blog.csdn.net/stormyk/article/details/89362078)  
[HTTPS vs SSH in git](https://ourtechroom.com/tech/https-vs-ssh-in-git/)  
[使用 Ed25519 算法生成你的 SSH 密钥](https://zhuanlan.zhihu.com/p/110413836)  
[详解：为GitHub、Gitlab账号同时添加、管理多个SSH-Key](https://blog.csdn.net/qq_35658349/article/details/103334343)  
[How to set SSH on GitHub using Ed25519 algorithm in Colab?](https://stackoverflow.com/questions/67660585/how-to-set-ssh-on-github-using-ed25519-algorithm-in-colab)
