# hexo-page
## 命令行
具体查[此链接](https://hexo.io/zh-cn/docs/commands)
### 新建文章
```
$ hexo new [layout] <title>
```
### 生成文件
```
$ hexo generate
$ hexo generate -d --deploy /// 生成后立即部署
$ hexo generate -w --watch // 监视文件变动
```
### 发表草稿
```
$ hexo publish [layout] <filename>
```
### 启动服务器
```
$ hexo server
```
### 清除缓存文件和已生成的静态文件
```
hexo clean
```