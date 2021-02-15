---
title: 如何用Hexo在GitHub搭建博客并实现Travis CI自动部署
---
Hexo是一个博客框架，使用简单，我们只需要关注MarkDown文件的书写，它会帮我们渲染页面；GitHub是一个代码托管平台，现在被微软收购，号称世界最大最先进的开发平台，简单说就是你写的代码可以免费托管在它的服务器上；Travis可以帮你完成GitHub上代码的测试和发布，重点是我们可以免费使用它。

### 安装hexo

你需要先安装以下两个软件：
* 安装[node.js](https://nodejs.org/zh-cn/)（建议版本12.0及以上）
* 安装[git](https://git-scm.com/)

完成以上安装之后执行命令安装hexo，
``` bash
$ npm install -g hexo-cli
```
注意：如果出现```EACCES```权限错误，按[官方文档](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)解决。

等待安装完成之后，可以通过两种方式执行hexo指令：
1. ``` npx hexo <command>```
2. 添加环境变量，将Hexo所在的目录下的```node_modules```添加到环境变量之中即可直接使用```hexo <command>```

### 生成博客
``` bash
$ npx hexo init <folder>
$ cd <folder>
$ npm install
$ npx hexo server
```
执行完以上命令之后可以通过本地浏览器访问[http://localhost:4000]()

### 部署代码
1. 你需要在[GitHub](https://github.com/)上创建账号，并新建一个 repository。你的 repository 应该直接命名为 <你的 GitHub 用户名>.github.io。
2. 将你的 Hexo 站点文件夹推送到 repository 中。不要将public目录推送到repository 中，检查.gitignore文件中是否包含 public 一行，如果没有请加上。
3. 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。
4. 前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。
5. GitHub 新建 [Personal Access Token](https://github.com/settings/tokens)，只勾选 repo 的权限并生成一个新的 Token。
6. 回到 [Travis CI](https://github.com/marketplace/travis-ci)，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 ```GH_TOKEN```，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存。
7. 在你的 Hexo 站点文件夹中新建一个 .travis.yml 文件：
``` bash
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```
8. 将 .travis.yml 推送到 repository 中。Travis CI 应该会自动开始运行，并将生成的文件推送到同一 repository 下的 gh-pages 分支下
9. 在 GitHub 中前往你的 repository 的设置页面，修改 GitHub Pages 的部署分支为 gh-pages。




