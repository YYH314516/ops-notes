# GitHub 从零上手（运维学习记录）

> 写给完全没碰过 git/GitHub 的人。照着做一遍，大概 20 分钟，之后每次只要 3 条命令。
> **今天下午的 D2 不需要 GitHub。** 先在本地写笔记，回来再推。

---

## 一、先搞清楚两个东西

- **git**：装在你电脑上的版本管理工具，负责"记录每次改了什么"。**你已经装好了**（2.54.0）。
- **GitHub**：一个网站，存你的仓库。相当于把本地的 `.git` 那份历史备份到网上，顺便让面试官能看到。

日常流程永远是这三步：**改文件 → `git add` / `git commit` 存到本地 → `git push` 传到 GitHub**。

---

## 二、我需要你先确认三件事

1. **有没有 GitHub 账号？** 没有就去 https://github.com 用邮箱注册，用户名建议用拼音或英文（面试会看），别用乱码。
2. **用户名和注册邮箱**是多少？告诉我，我把本地仓库的提交身份改成正确的（现在是占位值 `YYH` / `yyh@example.com`，不改也能用，但提交记录上显示的身份不对）。
3. 这台电脑上是不是**同一个 GitHub 账号**？如果之后在另一台电脑上推，两台都用同一个账号就行。

---

## 三、第一步：在 GitHub 上建一个空仓库

1. 登录 GitHub，右上角 `+` → **New repository**
2. 填：
   - **Repository name**：`ops-notes`（或 `linux-notes`，英文小写，别用中文）
   - **Description**：`运维学习笔记`（可留空）
   - **Public / Private**：**选 Public**。公开仓库免费、面试能看，笔记里别放密码和私钥就行。（练习机密码这种不要提交）
   - 下面三个勾（Add README / .gitignore / license）**都不要勾**，本地已经有了，勾了会冲突
3. 点 **Create repository**，进去后会看到一页命令提示，先不管它。

---

## 四、第二步：让本机 git 认识你的 GitHub 账号

### 推荐做法：用 Git Credential Manager 弹浏览器登录

你的 Git 已经自带 GCM，最省事。执行下面两条，把用户名邮箱换成你自己的：

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的注册邮箱"
```

然后**第一次 push 的时候**会自动弹一个窗口，让你选 GitHub → 浏览器里点授权，之后就不用再输了。

> 如果没弹窗、或者弹出来是让你输密码：GitHub 早就不认账号密码了，见下面「PAT 方案」。

### PAT 方案（弹窗不工作时的备选）

1. GitHub 右上角头像 → **Settings** → 左侧最下面 **Developer settings**
2. **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**
3. Note 随便写 `win-pc`；Expiration 选 90 天；勾选 **repo** 这一个权限就够
4. 生成后**立刻复制**（页面关了就再也看不到）
5. push 时用户名填 GitHub 用户名，密码位置粘贴这串 token

### SSH 方案（可选，更安全但步骤多，先跳过）

以后觉得每次弹窗烦再弄，现在不需要。

---

## 五、第三步：把本地和 GitHub 连起来

本地仓库我已经建好了（在 `C:\Users\YYH\Desktop\c规划`）。在 **Git Bash** 里执行：

```bash
cd ~/Desktop/c规划

# 1. 先提交一次，把现有文件存进本地历史
git add .
git commit -m "初始化：学习规划与 D2 任务单"

# 2. 关联远程仓库：把 URL 里的 用户名/仓库名 换成你自己的
git remote add origin https://github.com/你的用户名/ops-notes.git

# 3. 首次推送（会弹登录窗口）
git push -u origin main
```

成功的样子：最后一行是 `branch 'main' set up to track 'origin/main'`，然后回到 GitHub 页面刷新，能看到文件。

**如果报 `remote origin already exists`**：说明之前加过，改用
```bash
git remote set-url origin https://github.com/你的用户名/ops-notes.git
```

**如果报 `failed to push some refs` / `rejected`**：说明远程不是空的（可能你建仓库时勾了 README）。执行：
```bash
git pull origin main --allow-unrelated-histories
# 有冲突就按提示改一下，然后 git add . && git commit
git push -u origin main
```

---

## 六、之后每天怎么推（这才是重点）

每天做完练习，在 `notes/` 下新建一个当天的笔记，例如 `notes/2026-11-13-D2.md`，然后：

```bash
cd ~/Desktop/c规划
git add .
git commit -m "D2 用户权限：SGID 目录与 sudo 免密"
git push
```

就这三条。`-m` 后面写这次干了什么，别写 "update" 这种没信息的。

**忘了写 -m 会怎样**：会打开一个编辑器，一般是 vim。`i` 进入编辑，写完按 `Esc`，输 `:wq` 回车。嫌烦就用 `git commit -m "..."`。

---

## 七、一台电脑一份配置（下午换电脑的情况）

另一台电脑上要做的：
1. 装 Git（https://git-scm.com/download/win ，一路默认）
2. 配置身份：`git config --global user.name` / `user.email`
3. **把仓库 clone 下来**，不要重新 init：
   ```bash
   cd ~/Desktop
   git clone https://github.com/你的用户名/ops-notes.git c规划
   ```
   首次 clone 也会弹登录窗口。

**关键规则：永远先 `git pull` 再动手。** 两台电脑换着用时，开工前先拉一次，不然会冲突。

---

## 八、常见报错对照

| 报错 | 意思 | 怎么办 |
|---|---|---|
| `Permission denied (publickey)` | 走 SSH 但没配密钥 | 改用 HTTPS 的 URL（`git remote set-url`） |
| `Authentication failed` | 用了账号密码而不是 token | 用 PAT，或用 GCM 弹窗登录 |
| `Support for password authentication was removed` | 同上 | 同上 |
| `Nothing to commit` | 没改动或没 `git add` | 先 `git add .` |
| `Updates were rejected` | 远程有你本地没有的提交 | 先 `git pull`，再 push |
| `fatal: not a git repository` | 不在仓库目录 | `cd ~/Desktop/c规划` |
| push 上去中文文件名乱码 | 终端编码显示问题 | GitHub 网页上看是正常的，不影响 |

---

## 九、这个仓库里不要放什么

- 任何密码、私钥、token（练习机密码这种只有你自己环境用的，也尽量别写，写 `见本地密码本`）
- 虚拟机镜像、ISO、大文件（GitHub 单文件超 100MB 直接拒）
- 老师给的、不属于你的代码

---

## 十、做不到就降级，别卡住

如果今天推不上去：

- **最低目标**：本地 `git commit` 成功。这只是本地存档，不需要网络。
- **次低目标**：笔记存进 `notes/` 目录，回头统一推。
- 推 GitHub 这件事**不影响你的技术成长**，只是秋招时的加分项和备份。D2 的练习本身更重要，别在这上面耗超过 30 分钟。
