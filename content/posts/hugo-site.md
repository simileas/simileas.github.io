+++
date = '2025-08-10'
title = 'Hugo 站点搭建'
categories = ['软件使用备忘']
file = 'hugo-site.md'
+++

Hugo 是一个非常流行的静态网站生成器，使用 Go 语言编写。

步骤如下：

1. **安装 Hugo**  
   你可以通过多种方式安装 Hugo，例如使用 Homebrew（macOS）、Chocolatey（Windows）或直接下载二进制文件。  
   例如，在 macOS 上，你可以运行以下命令：  
   ```bash
   brew install hugo
   ```

2. **创建新站点**
    使用以下命令创建一个新的 Hugo 站点：  
    ```bash
    hugo new site mysite
    cd mysite
    ```

3. **添加主题**  
   选择一个 Hugo 主题并将其添加到你的站点中。例如，使用 Git 克隆一个主题：  
   ```bash
   git init
   git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish
   ```

# 跑通站点

创建新页面：

```bash
hugo new posts/my-first-post.md
```

启动本地服务器预览站点：

```bash
hugo server -D -M
```

访问 `http://localhost:1313` 查看站点。

# 使用中遇到的问题

- **修改主题中的背景图、Logo 等**  
  背景图和 Logo 位于 `assets` 目录下，在配置项中直接引入文件名称即可；例如 homepageImage = "background.jpg"，包括 hero、用户头像等；
- **自定义配置**  
  编辑 `config.toml` 文件以自定义站点设置，例如标题、语言、主题等。在我的项目里使用的是 `hugo.toml` 文件；两者作用相同。
- **文章中的图片路径**  
  在 markdown 文件中插入图片时，确保使用正确的相对路径。例如，如果图片存放在 `static/images` 目录下，引用方式为 `![alt text](/images/your-image.jpg)`。
- **修改代码样式**  
  你可以编辑主题中的 CSS 文件来修改站点的外观。代码区域的样式通过 `hugo gen chromastyles` 命令生成，生成的样式直接写入站点的 CSS 文件（assets/css/custom.css）中。具体可参考 [Hugo 官方文档](https://gohugo.io/commands/hugo_gen_chromastyles/)。
- **显示数学公式**  
  如果你需要在站点中显示数学公式，可以使用 shortcodes `< katex >`，在 markdown 文件中引入 KaTeX 支持。

# 主题结构

我的主题是 Blowfish，站点的目录结构其实与 Hugo 的标准结构类似，主要包括以下几个重要目录：

- `archetypes/`：存放内容模板。
- `assets/`：存放静态资源，如 CSS、JavaScript 和图片。
- `data/`：存放数据文件，可以是 YAML、TOML 或 JSON 格式。
- `layouts/`：存放自定义布局文件。
- `static/`：存放静态文件，这些文件会直接复制到生成的站点中。

决定网站生成 html 文件的主要是 `layouts/` 目录下的模板文件，layouts 目录下通常包含以下子目录：  

- `partials/`：存放可重用的模板片段。
- `shortcodes/`：存放自定义短代码。

例如，站点使用了 tags 和 categories 这两个分类页面，有部分代码是在 `layouts/partials/taxonomy.html` 文件中定义的。

# 部署到 GitHub Pages

1. **生成静态文件**  
   运行以下命令生成静态文件：  
   ```bash
   hugo
   ```

2. **将生成的文件推送到 GitHub Pages**
   你可以使用 `gh-pages` 分支来托管你的站点。以下是一个简单的脚本示例：  
   ```bash
   cd public
   git init
   git remote add origin
   git add .
   git commit -m "Deploy Hugo site"
   git push -f origin main:gh-pages
   ```

3. **配置 GitHub Pages**
   在你的 GitHub 仓库设置中，选择 `gh-pages` 分支作为 GitHub Pages 的源。

# 自动化部署

你可以使用 GitHub Actions 来自动化部署 Hugo 站点。创建一个 `.github/workflows/deploy.yml` 文件，内容如下：

```yaml
name: Deploy Hugo site to GitHub Pages
on:
  push:
    branches:
      - main
  permissions:
    contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
      - name: Build Hugo site
        run: hugo --minify
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

这样，每次你将更改推送到 `main` 分支时，GitHub Actions 都会自动构建并部署你的 Hugo 站点到 GitHub Pages。

GITHUB_TOKEN 是 GitHub 自动为每个仓库创建的一个令牌，用于在工作流中进行身份验证。它允许你在工作流中执行诸如推送代码到仓库等操作，而无需手动创建和管理个人访问令牌。

## gh-pages 是如何工作的？

`gh-pages` 是 GitHub 提供的一个特殊分支，用于托管静态网站。你可以将生成的静态文件推送到这个分支，GitHub 会自动将其作为网站进行托管。访问方式通常是 `https://<username>.github.io/<repository>/`。

如果你的仓库名称是 `<username>.github.io`，那么你可以直接通过 `https://<username>.github.io/` 访问你的网站。

那么如何绑定自定义域名？

需要在你的仓库根目录下创建一个名为 `CNAME` 的文件，文件内容是你的自定义域名，例如：

```
www.example.com
```

然后在你的域名提供商处将该域名的 DNS 记录指向 GitHub Pages 的 IP 地址即可。GitHub 提供了详细的[文档](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)来指导你完成这个过程。
