# WeWorkFinancePush
企业微信会话存档消息内容推送，把获取回来的消息通过Http向订阅者推送。
---

### 项目起源
```
  由于企业微信的会话存档没有公开接口读取，也不提供主动消息内容推送（官方只提供消息事件推送，但是里面不包含内容），企业微信官方只提供C和Java的SDK，对其他语言非常不友好。
  我在PHP的项目中就遇到了这个问题，尽管PHP有大神写了扩展，但是对于小白来说编译PHP扩展还是很麻烦的，而且这个过程中还会遇到各种各样的奇葩问题很浪费时间。
  于是我就突发奇想如果我做一个独立的服务把读取会话存档的消息变成Http请求向订阅者主动推送，那么就很方便了，这样所有的语言都通用再也不需要管对接企微官方SDK的问题。
  最终我选择了用.NET来写这个服务，.NET可以很方便的支持跨平台，Windows/Linux/MacOS都能支持。
```

### 流程
![Image](https://user-images.githubusercontent.com/5276634/241429809-7941b972-6c80-4c72-a1aa-af34dca50ce6.jpg)


### 推送方式及验签
项目 | 说明
---|---
推送方式| Http POST
数据格式 | application/x-www-form-urlencoded
字符编码 | UTF8
签名算法 | MD5

推送消息的时候会带数据签名sign（所有POST数据按字典序使用URL键值对的格式拼接成字符串然后拼接&key=xxxx进行MD5）

注意以下重要规则：

    ◆ 参数名ASCII码从小到大排序（字典序）
    ◆ 参数名区分大小写；
    ◆ 验证推送消息通知的签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验

### 数据内容说明
字段 | 说明
---|---
nonce_str | 随机字符
sign | 数据的MD5签名
messages | 解密后的消息内容（JSON字符串）

messages 样例：
```
[
        {
                "text": {
                        "content": "测试内容1"
                },
                "msgid": "9859594715939472242_1683115009861_external",
                "action": "send",
                "from": "wm4jvMCgAA7h7-G19NZKC0v5Yfs8KYMg",
                "tolist": [
                        "XiMenDaShu",
                        "wm4jvMCgAAJsUJHBuncwzla1sQ90w2Nw"
                ],
                "roomid": "wr4jvMCgAAja65ebFF3yM83lqFk42PuQ",
                "msgtime": 1683115005305,
                "msgtype": "text"
        },
        {
                "text": {
                        "content": "测试内容2"
                },
                "msgid": "1650101850641329947_1683115014029_external",
                "action": "send",
                "from": "wm4jvMCgAA7h7-G19NZKC0v5Yfs8KYMg",
                "tolist": [
                        "XiMenDaShu",
                        "wm4jvMCgAAJsUJHBuncwzla1sQ90w2Nw"
                ],
                "roomid": "wr4jvMCgAAja65ebFF3yM83lqFk42PuQ",
                "msgtime": 1683115009206,
                "msgtype": "text"
        }
]
```
详细格式说明请参考企业微信官方文档：
https://developer.work.weixin.qq.com/document/path/91774


### 安装及配置
#### 1、安装.net 6运行环境
```  
微软官方下载安装地址： 
  https://dotnet.microsoft.com/en-us/download/dotnet/6.0

CentOS 7示例： 
  sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
  sudo yum install dotnet-runtime-6.0

```

#### 2、配置企业微信corpId及Secret等
```
打开config/wework.json

{
  "corpId":"", //这里写企业ID
  "secret":"", //这里写会话存档秘钥
  "keyIndex":1  //消息加密公钥版本号（在企微后台配置会话内容存档那有显示）
}

消息解密私钥 config/private.pem
注意：私钥是 -----BEGIN PRIVATE KEY-----开头的

```

#### 3、配置订阅者信息
```
打开config/subscribe.json
[
  {
    "callback":"http://xxxxxxx.xxxxx.com/xxx/xxx",  //回调通知地址
    "key":"23456788754334567834"   //回调数据签名key
  }
//可以配置多个订阅者
]
```

#### 4、运行
##### Windows
在命令行下运行WeWorkFinancePush.exe

##### Linux
```
dotnet WeWorkFinancePush.dll

或者

./start.sh 


```
