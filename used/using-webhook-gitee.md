大型项目协同开发常使用Git，渐渐的一些中小型的公司开始效仿，现如今成为一个普遍使用的代码仓库，为了满足开发需求，git仓库功能也逐渐多样化，如静态网页托管、代码质量分析、WebHook等。今天主要来讲讲WebHook的使用。

- ##### WebHook

  简单来说，WebHook就是一个接收HTTP 请求的URL

- ##### 用途

  更新客户端，在资源新建或者更新时提供更新的、指定的数据。

- ##### 使用场景

  本文案例是在git代码更新或提交时，应用服务器更新代码，实现自动部署

<br>



### 一、Tomcat服务器配置

这里我们要实现远程http调用之后可以启动shell脚本实现代码的`git pull`操作部署环境为tomcat，当然实现自动部署，开启了tomcat自动部署模式。

```bash
# 进入tomcat配置目录
cd /home/software/tomcat-9.0/conf
# 修改配置文件
vim server.xml
```

修改内容，添加一行，设置`reloadable="true"`自动更新Tomcat容器：

```xml
<Server>  
    <Service>
        <Engine> 
            <Host>
                <!-- 这里就是自己添加的标签，修改默认访问路径 -->
                <Context path="" reloadable="true" docBase="../webapps/index-self"/>
            </Host>
        </Engine>
    </Service>
</Server>

```

这里省略了其他配置，修改时添加一行当指定的位置，不要删除其他内容，不要删除其他内容，不要删除其他内容

<br>



### 二、安装环境

本文主要是讲述自动部署过程，使用的gitee自动部署工具：[GiteeWebHook](https://gitee.com/qiushunji/GiteeWebHook)

#### 1、克隆代码

```bash
git clone https://gitee.com/qiushunji/GiteeWebHook.git
#或
git clone git@gitee.com:qiushunji/GiteeWebHook.git
```

<br>



#### 2、安装服务

```
npm install
```

**注意：`-bash: npm: command not found`表示没有安装node.js，请事先[安装Node.js](http://zayl.top/about-install/#/docs/linux-install-nodejs)**

<br>



#### 3、修改配置文件config.js

```bash
cd /home/software/GiteeWebHook
vim config.js
```

```js
module.exports = {
        os: "linux",                            //系统类型：windows\linux
        path: "/gitee/webhook",                 //WebHook POST路径
        secret: "123456",                       //请求密码
        port: 1314                              //WEB Hook服务端口号
};

```

<br>



#### 4、配置命令脚本，将脚本添加至cmd目录，脚本名称为:仓库名称.sh。WEB_PATH的值需根据实际项目位置设定。

```bash
#!/bin/sh

WEB_PATH='/home/software/tomcat-9.0/webapps'
WEB_USER='root'
WEB_USERGROUP='root'

echo "Start deployment"
cd $WEB_PATH
echo "pulling source code..."
git reset --hard origin/master
git clean -f
git pull
git checkout master
echo "changing permissions..."
chown -R $WEB_USER:$WEB_USERGROUP $WEB_PATH
echo "Finished."
```

<br>



#### 5、启动

```bash
cd /home/software/GiteeWebHook
pm2 start pm2.json
```

**注意：`-bash: pm2: command not found`需要安装pm2并配置软连接**

```bash
npm install -g pm2
pm2 install pm2-intercom
```

如果第二条pm2命令失败，则配置软连接

```
ln -s /home/software/nodejs/bin/pm2 /usr/local/bin
```

再执行`pm2 install pm2-intercom`

<br>



### 三、Gitee添加WebHook

[添加WebHook](https://gitee.com/help/articles/4184#article-header0)

URL：`http://www.xxx.com:1314/gitee/webhook`

密钥key：`123456`

点击激活即可看到测试结果：

成功返回：{"ok":true}

<br>



### 附：

1、访问不到端口，你可能需要打开防火墙

```bash
firewall-cmd --zone=public --add-port=1314/tcp --permanent
firewall-cmd --reload
firewall-cmd --zone=public --query-port=1314/tcp
```

<br>

2、常用命令

```bash
# 监测状态
pm2 monit
# 查看列表
pm2 list
# 启动
pm2 start pm2.json
# 重启
pm2 restart pm2.json
# 停止
pm2 stop pm2.json
# 杀死PM2（出现某些情况启动失败）
pm2 kill
```

<br>



3、原作者使用的版本有bug，请查看你使用的这个版本是否存在这个问题。

问题查看：https://gitee.com/geshuyong/GiteeWebHook/pulls/1/files

描述：`os`修改为`config.os`

```js
- if(os=="linux"){
+ if(config.os=="linux"){
```

