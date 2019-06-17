# hexo and next

下载blog
```bash
$ git clone git@github.com:liuxingjun/blog.git
```

下载主题

```bash
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

下载npm依赖
```bash
$ npm install
```

设置别名
```bash
$ alias hexo=hexo --config=source/_data/next.yml
```

生成静态文件(generate)并监视(watch)文件变动
```bash
$ hexo g -w
```

启动服务器(server)使用静态(static)文件
```bash
$ hexo server -s
```

部署网站(deploy)
```bash
$ hexo d
```