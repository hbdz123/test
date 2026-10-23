# hello world 

> 核心一句话：**开着代理 → `git clone --depth 1 <网址>` → 完成**

---

## ⚠️ 前提：代理必须开着

之前 clone 失败（`Connection was reset`）就是代理没开。

**验证是否通了**（PowerShell 里跑）：
```
ssh -T git@github.com
```
- 看到 `Hi hbdz123!` → 通了 ✅
- 报连接失败 → 去开代理

你机器上的代理：`127.0.0.1:7897`

---

## 🚀 三条最常用命令

### 1. 下载仓库（SSH 方式，推荐）
```
git clone git@github.com:用户名/仓库名.git
```
会在当前目录建一个同名文件夹。

**放到指定位置**：先 `cd D:\我的项目` 再 clone。

### 2. 只要最新代码，不要历史（推荐）
```
git clone --depth 1 git@github.com:用户名/仓库名.git
```

| 方式 | ASu-skills 实测 |
|---|---|
| `git clone` | 完整历史，体积大 |
| `git clone --depth 1` | **23.6 MB / 6.4 秒** |

只想用代码、不想看历史提交，就用这个。

### 3. 之后更新到最新
```
cd 仓库名
git pull
```

---

## 📋 操作速查表

| 我想… | 命令 |
|---|---|
| 下载仓库 | `git clone <网址>` |
| 下载最新版省空间 | `git clone --depth 1 <网址>` |
| 更新到最新 | `git pull` |
| 看远程有哪些分支 | `git branch -r` |
| 切到某分支/版本 | `git checkout <分支名>` |
| 看最近提交 | `git log --oneline -5` |
| 看当前改动 | `git status` |
| 丢掉本地改动 | `git checkout -- .` |
| 看远程地址 | `git remote -v` |

---

## 🔗 网址怎么找

GitHub 仓库页 → 绿色 **Code** 按钮 → 复制地址。

**两种方式都可用**：

| 方式 | 地址格式 | 需要 |
|---|---|---|
| **SSH**（推荐） | `git@github.com:用户/仓库.git` | 只需首次配置（已完成） |
| HTTPS | `https://github.com/用户/仓库.git` | 下载免认证，push 需 token |

**Code 按钮 → SSH 标签** 可以复制 SSH 地址（显示为 `git@github.com:...`）。

---

## 🔑 SSH 配置（已完成，此节供备查）

### 我这台机器已完成的三件事

| 项目 | 状态 |
|---|---|
| SSH 密钥 | ✅ 已有 `C:\Users\LENOVO\.ssh\id_rsa`（RSA 4096位） |
| GitHub 公钥登记 | ✅ 已登记，`ssh -T git@github.com` 返回 `Hi hbdz123!` |
| git 身份 | ✅ `user.name = HBDZ123` / `user.email = 2376058934@qq.com` |

### 关键配置：`.ssh/config`

路径：`C:\Users\LENOVO\.ssh\config`

```
Host github.com
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```

**为什么需要**：默认 SSH 走 22 端口，在国内网络环境下经常 `Connection timed out`。
GitHub 官方提供 **443 端口**作为备用通道（`ssh.github.com`）。
加了这段配置后，写 `git@github.com:...` 会自动走 443，不用改任何命令。

### 验证 SSH 是否正常

```powershell
ssh -T git@github.com
```

成功输出：
```
Hi hbdz123! You've successfully authenticated, but GitHub does not provide shell access.
```

> 退出码是 1 属于正常现象，GitHub 不提供 shell，所以"拒绝"执行命令。

### 查看实际连的是哪个端口

```powershell
ssh -v -T git@github.com 2>&1 | Select-String "Connecting to|Connection established"
```

应显示：`Connecting to ssh.github.com [...] port 443`

### 换新电脑时怎么重建

1. **生成密钥**（如果没现成的）：
   ```powershell
   ssh-keygen -t ed25519 -C "你的邮箱"
   ```
   一路回车用默认路径，密码可留空。

2. **复制公钥**：
   ```powershell
   Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" | Set-Clipboard
   ```

3. **贴到 GitHub**：访问 https://github.com/settings/keys → New SSH key → 粘贴

4. **建 `.ssh/config`**（内容同上，把 `id_ed25519` 换成实际文件名）

5. **验证**：`ssh -T git@github.com`

---

## 🌿 提交自己的改动（push 流程）

```powershell
cd 仓库目录
git status                     # 看改了什么
git add .                      # 把所有改动加入暂存
git commit -m "说明这次改了什么"   # 拍快照
git push                       # 推到 GitHub
```

> SSH 已配好，**push 不需要输密码**。

### 给别人提 PR 的流程

```powershell
# 1. 先在 GitHub 网页上点 Fork（复制到自己账号）
# 2. 克隆自己那份
git clone git@github.com:HBDZ123/原仓库名.git
cd 原仓库名

# 3. 改代码
# 4. 提交
git add .
git commit -m "修复了xxx问题"
git push

# 5. 回 GitHub 网页点 Compare & pull request
```

---

## ❓ 常见报错

| 报错 | 原因 | 解决 |
|---|---|---|
| `Could not connect to server` | 代理没开或网络不通 | 开代理后重试 |
| `Connection was reset` | 同上，连接被重置 | 开代理后重试 |
| `Connection timed out`（端口22） | 22 端口被干扰 | 已有 config 走 443，若仍失败检查 config |
| `'git' 不是内部或外部命令` | git 不在 PATH | 检查 PATH 是否含 `D:\Git\cmd` |
| `fatal: not a git repository` | 当前目录不是仓库 | 先 `cd` 进仓库目录 |
| `Permission denied (publickey)` | 公钥没贴到 GitHub | 重新执行上面的"贴到 GitHub"步骤 |
| `Host key verification failed` | known_hosts 里的旧记录冲突 | 删除 `C:\Users\LENOVO\.ssh\known_hosts` 里对应行 |

---

## 📌 我的环境

| 项目 | 值 |
|---|---|
| Git 版本 | 2.55.0.windows.1（便携版） |
| Git 位置 | `D:\Git\` |
| 代理 | `127.0.0.1:7897` |
| SSH 密钥 | `C:\Users\LENOVO\.ssh\id_rsa`（RSA 4096位） |
| SSH 配置 | `C:\Users\LENOVO\.ssh\config`（走 443 端口） |
| GitHub 账号 | HBDZ123 |
| 提交邮箱 | 2376058934@qq.com |
| 技能目录 | `D:\deepseekH\.dsh\skills\asu\` |

---

## 💡 下载 vs 贡献

- **只想用别人的代码** → `clone` 就够了
- **想提交自己的改动、提 PR** → 已配置完毕，直接用 `git push` 即可

---

## 🔧 其他认证方式：HTTPS + Token

如果哪天 SSH 出问题，还可以用 HTTPS + Personal Access Token：

1. 访问 https://github.com/settings/tokens
2. **Generate new token (classic)**
3. 勾选 `repo` 权限，生成后**立刻复制**（只显示一次）
4. push 时用户名填 `HBDZ123`，密码栏粘贴 token
5. Windows 会存进凭据管理器，之后不用再输

---

*创建于 2026-10-22 | 更新于 2026-10-22（补充 SSH 配置）*
