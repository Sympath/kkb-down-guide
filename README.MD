## 本仓库存在意义

搭建开课吧下载工具的环境向导，最终的产物流程如下

![20220816142645](https://raw.githubusercontent.com/Sympath/imgs/main/20220816142645.png)

## 应用架构

1. kkb-Web：用户端(谷歌插件)，包名，发起请求从而触发下载；这里要选择服务器，如果选择 local 开启时就会触发本地的 3000 端口(https://github.com/Sympath/kkb-down-web)
2. kkb-serve：服务端，接收请求，接收后进行下载【download-sh】仓库并输入依赖，执行课程下载流程（https://github.com/Sympath/download-serve）
3. download-sh：下载流程脚本（https://github.com/Sympath/download-sh）
4. kkb-download：核心下载模块（https://github.com/Sympath/kkb-download）
5. kkb-serve-deploy：https://github.com/Sympath/down-serve-deploy，服务端环境部署，用于在新的 centos 云服务器搭建 kkb-serve 环境

## 搭建步骤

1. 找一台远程服务器(centos7 系统)，下拉【kkb-serve-deploy】仓库

```
git clone git@github.com:Sympath/down-serve-deploy.git
```

2. 安装基础环境【百度云盘脚本上传 + ffmpeg 依赖 + kkb-serve 服务搭建】

2.1 安装 linux 百度云盘（这个需要获取授权码，所以没法自动化）安装完成后会输出一个地址，访问输出的的地址获取授权码，输入授权码进行验证即可

![image-20220905150524283](https://tva1.sinaimg.cn/large/e6c9d24ely1h5vqs4cx6qj219a0mi40y.jpg)

执行如下命令

```
cd down-serve-deploy && sudo chmod 777 ./ && nohup sh ./bypy-install.sh
```

2.2 执行核心脚本，会自动完成其他环境搭建和 kkb-serve 的服务启动

```
nohup sh ./init.sh
```

3. 客户端：下拉谷歌插件 https://github.com/Sympath/kkb-down-web代码，然后添加自己的服务器ip地址即可(搜索【Here's the Server】，按规则填写，选择框就会有对应选择项了)

```
git clone git@github.com:Sympath/kkb-down-web.git
```

## TODO

1. 完成短信验证码登陆接入
2. 剥离开课吧依赖，从而扩展支持各大平台接入 **checked**
3. 缓存机制，如果已经下载成功的课程，在下载流程中断之后的二次重启过程中，不要再下载；checked
4. 服务器快速部署能力 ==》 支持一个初始化的 linux 服务器直接执行脚本自动搭建 kkb-down-serve，及 kkb-down-core 运行环境 **checked**
   1. 配置 git 环境
   2. 搭建 kkb-down-core 运行环境：ffmpeg 安装
   3. 下拉 kkb-down-serve 仓库、安装依赖并 pm2 启动
5. 加下拉框，支持点击下拉显示目前已经上传的包列表
6. 终端连接：将下载的日志输出在谷歌插件面板中
7. 待实现服务器主动推送：完成时主动推消息
8. kkb-serve-deploy：linux 版百度云盘需要授权，授权过程是【1. `bypy info`触发授权 2. 访问输出的的地址获取授权码 3. 输入授权码进行验证】，这部分暂时需要手动处理
9. 复用依赖问题：每次下载都会安装一遍 kkb-down-core 的核心依赖，这个是否可以复用
10. 谷歌插件优化
    - pageAction 接入：针对开课吧网站才打开
11. 将所有的 shell 都放在 kkb-down-sh 中
12. 自动化执行：为了避免并行进程过多，对下载任务进行了拆分处理，每次只做 10 条下载任务；而这时就需要人为的看当前任务是否完成，完成则执行下一条，这点需要进行优化。思路如下
    1. 获取下一个脚本： 每个脚本执行完就记录下当前执行的序号，下一个脚本名称就是序号+1
    2. 执行时机：可以设定一个定时任务，定时 ps -ef 查询当前是否还存在 sh/config\*/index.sh 进程，不存在了就执行下一条即可
13. 常用命令记录：项目根目录下生成常用执行命令，参考【常用命令】模块 **checked**
14. upload-remote: 需要考虑完全没有上传过的情况
15. 账户结束标识：
    1. 记录脚本信息对象，存储信息如下
       - 有多少个 all 脚本：allNum
       - all 脚本对应信息对象
         - 当前 all 脚本的课程数 nohupNum
         - 当前 all 脚本的序号 serial
         - 当前 all 脚本的中止进程命令
    2. 定义完成课程数累加器 completeNum，初始为 0，课程脚本结束时将累加器+1，
    3. completeNum === allNohupNum 时，进行下一步
       1. 当所有都执行完了，即 allNum；则发送完成邮件
       2. 当没有执行完，执行下一个脚本; 执行 npm run build-sh${serial+1}
16. 单个课程下载：自动判断当前页是课程页还是视频页
