---
layout: post
title: "npm常用命令"
date: 2018-01-05 12:00:00
categories: js
comments: true
---

- 查看当前版本

```bash
npm --version
```

- 升级npm

```bash
npm i -g npm to update
```

- 安装与删除npm包

```bash
# 当前目录安装
npm install package-name

# 当前目录安装 并且在package.json写入文件中的devDependencies，开发依赖
npm install package-name --save-dev

# 指定版本
npm install package-name@1.2.3

# 全局安装
npm install -g package-name

# 示例
npm install uglify-js --global
npm install uglify-js -g

npm i uglify-js --save-dev
npm i uglify-js -D

npm i uglify-js@0.1.1

npm install git+https://isaacs@github.com/npm/npm.git
npm install git+ssh://git@github.com:npm/npm.git#v1.0.27
npm install bitbucket:mybitbucketuser/myproject

# 不下载devDependencies
npm install --only=production
# 当前目录删除
npm uninstall package-name

# 全局删除
npm uninstall package-name -g

npm uninstall express
npm uninstall express -g

# 全局更新
npm update -g
# 当前目录更新
npm update

npm update package_name
```

- 查找npm包

```bash
npm search express
```

- 初始化项目

```bash
npm init
```

- 列出已将安装的npm包

```bash
# 全局
npm list --global
# 全局 只列1层
npm list -g --depth=0

当前目录
npm ls
当前目录 production
npm ls --prod


cd /path/to/the/project
# 当前目录
npm ls
# 当前目录详情
npm ls -l

# 查看包的主页
npm home package_name
# 查看包的Github repo
npm repo package_name
```

- 查看是否有更新的包

```bash
# 当前目录
npm outdated

# 全局
npm outdated -g

npm outdated --prod
```

- 移除没有使用的包

```bash
npm prune

# 在 node_modules 中删除所有devDependencies的包
npm prune --production
```

- 移除重复

```bash
npm dedupe
```

- 列出缓存

```bash
npm cache ls
```

- 删除缓存

```bash
npm cache clean -f
```

- 开启自动完成

```bash
npm completion >> ~/.bashrc
npm completion >> ~/.zshrc
```
