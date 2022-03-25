# hexo and next

下载源码
```bash
git clone git@github.com:liuxingjun/liuxingjun.github.io.git blog
git submodule init
git submodule update #下载模版
```
or
```
git clone --recurse-submodules https://github.com/liuxingjun/liuxingjun.github.io.git blog
```

下载npm依赖
```bash
npm install
```

生成静态文件(generate)并监视(watch)文件变动
```bash
npx hexo g -w
```

启动服务器(server)使用静态(static)文件
```bash
npx hexo server -s
```
