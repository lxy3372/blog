title: "开始使用Hexo搭建博客"
date: 2015-03-26 06:33:49
tags: [蜕变,hexo,mac,blog]
---
告别wordpress之后，我一直在寻找代替品，试过复杂的Jetty，也试过其他基于LAMP的CMS，最终选择了Hexo，安装也不算复杂：
1. 安装Node.js
```bash
brew install node  #OS X 环境
```
2. 安装Hexo
```language
npm install -g hexo
```
3. 之后就可以在本地初始化博客了
```bash
hexo init blog
cd blog
#此间会提示你npm install 
npm install
```
4. 修改配置文件_config.yml 配置站点信息,这个比较简单

5. 生成新的文章
```bash
hexo new "标题" #会在source/_post 下生成对应的md文件，采用markdown编辑器编辑即可
```
6. 然后生成静态化站点
```bash
hexo generate
```
7. 本地测试
```bash
hexo server
```
8. 如果需要发布的Github上，先在配置文件中配置好Github信息，这里需要现在github账号下创建项目blog（注意不要创建README）
```bash
deploy:
      type: github
      repository: https://github.com/lxy3372/blog.git
      branch: gh-pages
```
9. 发布
```bash
hexo deploy
hexo deploy --generate
```


- - -
杂记：
   在折腾期间也遇到了一些问题，比如编辑_config.yml的时候，如果用vim编辑的话，在启动server的时候会出现格式错误的问题，而直接用文本编辑则不会出现这种问题，这可能跟我vim的配置有些关系。该文也是我参考了其他朋友的文章，按自己搭建的顺序写的，也算是重复造轮子了，不过当Markdown练练手也是不错的。

