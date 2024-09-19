+++
title = "Tasker配合ntfy接收通知"

date = 2024-09-19

[taxonomies]
tags = ["Tasker", "ntfy", "notification"]
categories = ["technology"]
+++

[`ntfy`](https://github.com/binwiederhier/ntfy)是一个在设备间传递消息的工具。可以直接使用官方提供的服务，也可以自建。类似的工具还有很多，例如[`gotify`](https://gotify.net/docs/index)等等。这里以`ntfy`为例，其他工具也大多支持`GET`或`PUT`的方式发送和接收消息。

`Tasker`是一个在安卓设备上非常流行的自动化工具，可以发送`GET`请求与`ntfy`服务器通讯来接收消息。

## `ntfy`发送和接收消息

### 发送消息

`ntfy`支持众多的发送消息方式，这里以`PUT json`为例。更多内容参考<https://docs.ntfy.sh/publish/>。

注意`URL`中不包含`topic`，而是放在了`JSON`中。文档中的示例：

```js
fetch('https://ntfy.sh', {
    method: 'POST',
    body: JSON.stringify({
        "topic": "mytopic",
        "message": "Disk space is low at 5.1 GB",
        "title": "Low disk space alert",
        "tags": ["warning","cd"],
        "priority": 4,
        "attach": "https://filesrv.lan/space.jpg",
        "filename": "diskspace.jpg",
        "click": "https://homecamera.lan/xasds1h2xsSsa/",
        "actions": [{ "action": "view", "label": "Admin panel", "url": "https://filesrv.lan/admin" }]
    })
})
```

### 接收消息

这里和发送消息类似。可以使用`Javascript`中的`fetch`函数发送`GET`请求。返回的数据为`JSON`格式。

## 利用`Tasker`接收消息

由于涉及到从网络上获取信息，所以存在比较高的失败可能性，因此需要添加大量的错误处理逻辑。为了整体流程清晰，强烈建议将逻辑完全写在一个`Javascript`脚本中。

```js
// ntfy的每个消息都有一个ID。利用这个ID可以只获取最新的消息。

let lastNotification;
try {
    lastNotification = global('%NotificationID');
} catch (e) {
    lastNotification = 'notificationID';
    console.log('Run in browser');
}
const ntfyUrl = 'https://ntfy.sh/mytopic/json?poll=1&since=' + lastNotification;

// 日志输出。根据情况输出到浏览器控制台或者手机日志文件。
const taskerLog = function (str) {
    const isOnMobile = typeof flash === 'function';
    if (isOnMobile) {
        writeFile('ntfy.log', str + '\n', true);
    } else {
        console.log(str);
    }
}

// fetch notifications from ntfy
const fetchNotifications = function () {
    taskerLog('开始查询URL: ' + ntfyUrl);
    fetch(ntfyUrl)
        .then(response => {
            taskerLog('Fetch response status:');
            taskerLog(response.status);
            return response.text()
        })
        .then(data => {
            if (data) {
                const lines = data.trim().split('\n');
                lines.forEach(line => {
                    let latestID;
                    let latestTime = 0;
                    try {
                        const notification = JSON.parse(line);
                        const title = notification.title || "New Notification";
                        const message = notification.message || "You have a new notification.";
                        const id = notification.id;
                        const time = notification.time;
                        if (time > latestTime) {
                            latestID = id;
                        }

                        taskerLog(title);
                        taskerLog(message);

                        // 将得到的信息传递给另一个任务“通知栏提醒”。
                        let result = performTask(
                            '通知栏提醒', local('%priority'), title, message
                        );
                        taskerLog(result);

                        setGlobal('%NotificationID', latestID);
                    } catch (e) {
                        throw new Error('Error parsing JSON' + e + 'Line:' + line);
                    }
                })
            } else {
                taskerLog('No new notifications.');
            }
        })
        .then(r => exit())
        .catch(error => {
            taskerLog('Error fetching notifications: ' + error);
        });
    taskerLog('查询结束');
}

fetchNotifications();
```

由于`Tasker`中的`Javascript`不支持直接发送通知栏提醒，所以需要另外创建一个`Task`。利用`performTask`启动`Task`并传递相关参数。

“通知栏提醒”这个`Task`中仅需要包含`Notify`这一个`Action`。`Action`中可以使用`%par1`和`%par2`这样的变量获取前面传递的参数。`Notify`中的具体内容可以根据情况填写。
