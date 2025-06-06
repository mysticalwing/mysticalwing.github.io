+++
date = '2025-06-06T17:43:14+08:00'
draft = false
title = 'Hugo+Github 部署个人静态博客'
+++

_note: 文章内容大部分由 gpt 生成，作者仅做了少量修改_

下面是 Hugo + PaperMod 博客部署到 GitHub Pages 的自动化流程，**从创建到访问网站**全流程讲解。

---

## 🗂️ 项目结构规划

你将有两个选项部署 Hugo 博客：

| 方式                  | 描述                                                           | 推荐用途                                                       |
| --------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- |
| ✅ **方式 A（推荐）** | 使用专门仓库，如 `mysticalwing.github.io` 作为 GitHub Pages 源 | 博客位于根域名：`https://mysticalwing.github.io/`              |
| 方式 B                | 使用 `my-hugo-blog` 仓库，部署到 `gh-pages`                    | 博客位于子路径：`https://mysticalwing.github.io/my-hugo-blog/` |

我们采用 **方式 A（部署到根域名）**。

---

## ✅ 全流程指南

---

### ① 安装 Hugo（仅本地操作时需要）

#### macOS:

```bash
brew install hugo
```

#### Windows:

```bash
scoop install hugo
```

#### 检查版本：

```bash
hugo version
```

---

### ② 创建 Hugo 博客项目

```bash
hugo new site mysticalwing.github.io
cd mysticalwing.github.io
# delete hugo.toml and use config.toml
rm hugo.toml
```

---

### ③ 添加主题 PaperMod

```bash
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

---

### ④ 配置 `config.toml`

在根目录下新建或编辑 `config.toml`：

```toml
baseURL = 'https://mysticalwing.github.io/'
languageCode = 'en-us'
title = 'MysticalWing Blog'
theme = 'PaperMod'
pagination.pagerSize = 10
enableRobotsTXT = true

[params]
  homeInfoParams = { Title = "你好 👋", Content = "欢迎来到我的博客！" }
  ShowReadingTime = true
  ShowShareButtons = true
  ShowPostNavLinks = true
  ShowBreadCrumbs = true

[[menu.main]]
  name = "主页"
  url = "/"
  weight = 1

[[menu.main]]
  name = "归档"
  url = "/archives/"
  weight = 2

[[menu.main]]
  name = "标签"
  url = "/tags/"
  weight = 3
```

---

### ⑤ 创建文章示例

```bash
hugo new posts/hello-world.md
```

编辑 `content/posts/hello-world.md`，改成如下内容(重点是将 draft: true -> false)：

```markdown
+++
title: "Hello World"
date: 2025-06-06T15:00:00+08:00
draft: false
+++

这是我使用 Hugo 和 PaperMod 搭建的第一篇博客！
```

---

### ⑥ 添加 GitHub Actions 工作流

在目录下创建：

```
.github/workflows/deploy.yml
```

内容如下：

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main # 当推送 main 分支时触发

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive # very important !!!

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
          cname: mysticalwing.github.io
```

---

### ⑦ 推送到 GitHub 仓库

1. 在 GitHub 创建仓库：`mysticalwing.github.io`（**必须是这个名字**）
2. 本地执行：

```bash
git remote add origin https://github.com/mysticalwing/mysticalwing.github.io.git
git add .
git commit -m "init Hugo + PaperMod"
git push -u origin main
```

---

### ⑧ 首次手动创建 `gh-pages` 分支（可选）

你可以提前创建空的 gh-pages 分支：

```bash
git checkout --orphan gh-pages
git rm -rf .
echo "Init Pages" > README.md
git add .
git commit -m "init gh-pages"
git push origin gh-pages
git checkout main
```

（GitHub Actions 也可以自动创建）

---

### ⑨ 配置 GitHub Pages

进入仓库 `Settings → Pages`：

- Source: `gh-pages`
- Branch: `/ (root)`
- 自动保存

---

### ✅ 部署成功后访问网站：

👉 [https://mysticalwing.github.io](https://mysticalwing.github.io)

---

### 🔁 后续更新流程

每次你：

```bash
hugo new posts/xxx.md  # 添加文章
# 编辑后设置 draft: false

git add .
git commit -m "添加新文章"
git push
```

GitHub Actions 自动构建并发布。

---

### ✅ 可选：配置自定义域名（可跳过）

如你有自己的域名（如 `blog.mysticalwing.com`），你只需在仓库根目录添加：

```
static/CNAME
```

内容为：

```
blog.mysticalwing.com
```

然后在域名解析中添加 CNAME 指向 `mysticalwing.github.io`。

---

## 🎉 小结

| 步骤     | 内容                                   |
| -------- | -------------------------------------- |
| 本地创建 | `hugo new site mysticalwing.github.io` |
| 添加主题 | `git submodule add`                    |
| 配置     | `config.toml` 和 `deploy.yml`          |
| 推送     | 到 `main` 分支，自动部署到 `gh-pages`  |
| 访问地址 | `https://mysticalwing.github.io`       |

---
