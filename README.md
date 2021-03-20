# qidianHelper
起点中文网在线时长经验值奖励自动领取，起点API

## release
Beta 1.6 https://github.com/creative-kz/qidianHelper/releases/tag/f6fc598

## 缘由
个人比较喜欢在起点读书，但是推荐票需要经验值等级23级才能获得3张实在有点顶不住，查了一下只有网页版每天在线时长120个经验值领取比较靠谱，但是领取时间一共要185分钟，所以就查了一下代码做了一个自动领取经验值的Chrome插件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210321065534470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbmRzb21laG9uZXk=,size_16,color_FFFFFF,t_70)

## 原理
原理很简单，到时间直接向起点api发送请求即可领取经验值，但是起点最近又更新完善了安全措施以后会验证所有请求的ip地址导致无法将这个程序扔到服务器上（起点新电脑登录有很多步的验证），所以只能做成插件在本地跑了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210321070158718.png)

可以看到URL就是 https://my.qidian.com/ajax/Score/Receive 再加两个参数：token和referobject，也就是第几个经验值，那么就很好办了，做了个倒计时的function以及一个get的function。主要代码如下：

链接起点api的函数：writeLog和stop是我的日志函数以及停止倒计时的函数
```javascript
const sourceURL = "https://my.qidian.com/ajax/Score/Receive";

// 获取经验值
function getEXP(callback){
    writeLog("尝试领取第 " + referObject + " 个在线时长奖励");
    $.get(sourceURL, {"_csrfToken": token, "referObject": referObject}, function(data){
        console.log(data);
        if(data.msg != null){
            if(data.msg == successMsg){
                console.log("领取成功");
                writeLog("领取成功：第 " + referObject + " 个在线时长成功领取");
                callback(0);
            }else{
                console.log(data.msg);
                stop("失败：原因" + data.msg + "，第 " + referObject + " 个在线时长未领取");
                callback(10);
                
            }
        }else{
            console.log("账户失效，无法访问");
            stop("失败：原因账户失效，请重新登录，程序已暂停");
            
        }
        
    // 网络错误
    }).fail(function(err){
        stop("失败：原因网络错误：" + err.status + ": " + err.statusText);
        console.log("网络错误 " + err.status + ": " + err.statusText);
        reconnect();
    });
}
```


倒计时的函数就很简单了，这边设置的时间间隔是1秒目的是为了更新我在popup页面的显示，refresh是我更新下一个倒计时的函数。
```javascript
// 领取倒计时
function countdown(){
    writeLog("计划于 " + timeleft + " 秒后领取在线时长奖励");
    if(p2countdown != null){
        clearTimeout(p2countdown);
        console.log("p2countdown: clear");
    }
    p2countdown = setInterval (function () {
        timeleft--;
        console.log("timeleft: " + timeleft);
        if(timeleft <= 0){
            clearTimeout(p2countdown);
            console.log("p2countdown: clear");
            
            getEXP(function(time){
                refresh(time);
                
            });
        }
    }, 1000);
}
```

## 几点说明：
1. 这个插件完全是在你前端浏览器上运行的，所以你不能关闭浏览器，但可以去访问其他页面，整体运行时间是185分钟，也就是3个多小时，你每天打开浏览器的时间应该就够了
2. 这个插件只会访问你起点的两个cookie：一个是你的token，一个是你的阅文账号，token的有效期大约在48-60小时，所以最好1-2天检查一下起点登录状态，插件里我也进行了自动检查账户有效性。

## 其它
个人兴趣（纯属闲的），又去获取了起点历史经验值记录然后做了一个可视化图表出来，觉得挺好看：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210321071957412.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbmRzb21laG9uZXk=,size_16,color_FFFFFF,t_70)

计划等以后有时间再更新更新的内容，测试测试起点其他的API，准备加一个每天23点20分自动投推荐票，因为我老是会忘，感觉浪费了挺可惜的

大家有什么想法欢迎留言或者QQ群交流：511101539
