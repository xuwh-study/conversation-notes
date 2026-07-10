# GitHub 使用指南

> 👤 你的账号：`xuwh-study` | 🔗 连接方式：SSH

---

## 一、基础概念

| 概念 | 说明 |
|------|------|
| **仓库（Repository）** | 存放代码的地方，简称 repo |
| **克隆（Clone）** | 把 GitHub 上的仓库下载到本地 |
| **提交（Commit）** | 保存一次代码快照 |
| **推送（Push）** | 把本地代码上传到 GitHub |
| **拉取（Pull）** | 把 GitHub 上的更新下载到本地 |
| **分支（Branch）** | 独立的开发线，互不影响 |

---

## 二、创建仓库

### 方式 A：在 GitHub 网页创建

1. 打开 https://github.com/new
2. 填写仓库名（英文，如 `my-project`）
3. 选择 Public（公开）或 Private（私有）
4. **不要勾选** "Add a README file"
5. 点击 Create repository

### 方式 B：从本地推送到 GitHub

```bash
# 1. 本地初始化
mkdir my-project
cd my-project
git init

# 2. 写点代码
echo "# My Project" > README.md

# 3. 提交
git add .
git commit -m "first commit"

# 4. 关联远程仓库并推送（把仓库名换成你的）
git remote add origin git@github.com:xuwh-study/my-project.git
git push -u origin master
```

---

## 三、日常操作流程

### 完整一条龙：

```bash
git status                    # 查看改了哪些文件
git add .                     # 把所有修改加入暂存区
git commit -m "修改说明"       # 提交，写上你改了什么
git push                      # 推送到 GitHub
```

### 只提交部分文件：

```bash
git add src/app.js            # 只加这一个文件
git commit -m "修复了登录bug"
git push
```

---

## 四、克隆别人的仓库

```bash
# 克隆自己的仓库
git clone git@github.com:xuwh-study/仓库名.git

# 克隆别人的仓库（用 HTTPS）
git clone https://github.com/作者/仓库名.git
```

---

## 五、拉取最新代码

如果你的仓库在另一台电脑上改了，或者和别人协作：

```bash
git pull    # 拉取最新代码并合并
```

---

## 六、常用命令速查

| 命令 | 作用 |
|------|------|
| `git status` | 查看当前状态 |
| `git add .` | 添加所有修改 |
| `git commit -m "消息"` | 提交 |
| `git push` | 推送到 GitHub |
| `git pull` | 从 GitHub 拉取 |
| `git clone 地址` | 克隆仓库 |
| `git log` | 查看提交历史 |
| `git log --oneline` | 简洁版提交历史 |
| `git diff` | 查看具体改了什么 |

---

## 七、写 commit 消息的好习惯

```bash
# ✅ 好的写法——一目了然
git commit -m "修复登录按钮点击无响应的问题"
git commit -m "添加用户头像上传功能"
git commit -m "更新README文档"

# ❌ 不好的写法
git commit -m "改了点东西"
git commit -m "111"
git commit -m "update"
```

---

## 八、.gitignore 文件

有些文件不想上传（如 `node_modules`、密码文件），在项目根目录创建 `.gitignore`：

```
node_modules/
.env
*.log
.vscode/
```

---

## 九、SSH 连接问题排查

```bash
# 测试连接
ssh -T git@github.com

# 如果连不上，手动加载密钥
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## 十、下一步可以学的

- **分支管理**：`git branch`、`git checkout -b`
- **Pull Request**：在 GitHub 网页上提交代码合并请求
- **GitHub Actions**：自动化测试和部署
- **Issues**：任务跟踪和 Bug 管理

当你需要的时候告诉我，我帮你深入讲解任何一个主题。