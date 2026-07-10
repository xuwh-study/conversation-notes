# Git 与 GitHub 连接配置记录

> 📅 日期：2026-07-10
> 👤 GitHub 用户：xuwh-study

## 一、检查 Git 安装

- **版本**：`git version 2.55.0.windows.2`
- **路径**：`C:\Program Files\Git\mingw64\bin\git.exe`

## 二、配置 Git 用户名和邮箱

```bash
git config --global user.name "xuwh-study"
git config --global user.email "106617960@qq.com"
```

## 三、生成 SSH 密钥

```bash
ssh-keygen -t ed25519 -C "106617960@qq.com"
```

- 私钥：`~/.ssh/id_ed25519`
- 公钥：`~/.ssh/id_ed25519.pub`

## 四、上传公钥到 GitHub

1. 打开 https://github.com/settings/keys
2. 点击 **New SSH Key**
3. 粘贴公钥内容

## 五、配置 SSH Config

`~/.ssh/config` 内容：

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

## 六、测试连接

```bash
ssh -T git@github.com
```

结果：✅ `Hi xuwh-study! You've successfully authenticated`

## 最终配置总结

| 配置项 | 详情 |
|--------|------|
| Git 用户名 | `xuwh-study` |
| Git 邮箱 | `106617960@qq.com` |
| 连接方式 | SSH（Ed25519） |
| GitHub 用户 | `xuwh-study` |