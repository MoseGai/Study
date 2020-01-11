##安装ElasticSearch插件

##一 Head插件介绍

elasticsearch-head是elasticsearch的一款可视化工具，依赖于node.js ，所以需要先安装node.js

## 二 安装Node.js

详情见文章【安装nodejs】

##三 安装Grunt

```python
#Grunt是基于Node.js的项目构建工具。它可以自动运行你所设定的任务 
npm install grunt -g
```

##四 下载Head

```python
#地址：<https://github.com/mobz/elasticsearch-head>，可以用git下载，或者下载zip
# 解压后切换到目录下
cd elasticsearch-head
# 通过npm安装依赖
npm install
#启动
npm run start
#在浏览器里打开
http://localhost:9100/

```

##五 配置跨域

修改 Elasticsearch 安装目录中config 文件夹下 elasticsearch.yml 文件，加入下面两行：

![image-20191204184459831](https://tva1.sinaimg.cn/large/006tNbRwgy1g9kvwosokij30re078mzy.jpg)

添加配置时，：后必须空格，不然启动闪退

```python
http.cors.enabled: true
http.cors.allow-origin: "*"
```



##六 查看

看到如下效果表示成功

![image-20191204185137497](https://tva1.sinaimg.cn/large/006tNbRwgy1g9kvxi7jh0j31e40dq0vp.jpg)