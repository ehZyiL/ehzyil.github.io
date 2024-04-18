



## discord 

discord pwd

```
atwMTXgsifFuhF5
```



COZE-BOT

```
1217821295866286170
5bb04194c289f469b67970eeb1eb35161bdd8a6012acd1c3ce043e783c610f89
```

MTIxNzgyMTI5NTg2NjI4NjE3MA.G8Q7Sp.8Qshj9B4w2yCIS8Dr8zBe9nQJ4gSynM9XAH584



COZE-BOT2

```
1218435135985156187
1f55281399cec951883b9e7b766f25ad739816e6f5de95c9dd21c8531f6c96c8
```

MTIxODQzNTEzNTk4NTE1NjE4Nw.Gl19yR.F-xV9WqLM3bXJn7qd1CuzVa1pkU3q7Rvxfbfqg



CDP-BOT

```
1217821830740578395
f5c5b46f7b7787fb9356cade76ad3355f2e327f9527cdcfbc7f22686deef61d1
```

```
https://discord.com/oauth2/authorize?client_id=1217821830740578395&permissions=8&scope=bot
```

MTIxNzgyMTgzMDc0MDU3ODM5NQ.GyCltX.OkTnna6WWTjg-u0m_AcqYszKPxWhT0ZqWhQDSs





服务器id

```
1202972464657858601
```



Authorization:

```
MTIwMjk2OTg4NTk3NDcyODcxNw.G5ZPp8.Bflh84OaP6RGdayDrIdJx60njqkWxLPToGhXqg
```

CHANNEL_ID

```
1202972464657858603
```

​	Authorization2:

```
MTIxODQzMDg0NDA1NDY3MTM3MQ.GNEsBT.uJBCeTtjGonbxUhTuoGu3C0MuXBICy_Zr5r5rU
```

```
{
    "type": 3,
    "nonce": "1218432326183157760",
    "guild_id": "1202972464657858601",
    "channel_id": "1202972464657858603",
    "message_flags": 0,
    "message_id": "1218432147707269191",
    "application_id": "1217821295866286170",
    "session_id": "395f4572e1cc34b85bbb4c9d85d40d88",
    "data": {
        "component_type": 2,
        "custom_id": "suggestions:How can I customize my experience with your AI assistance?"
    }
}
```



## coze-discord-proxy



```
version: '3.4'
 
services:
  coze-discord-proxy:
    image: deanxv/coze-discord-proxy:latest
    container_name: coze-discord-proxy
    restart: always
    ports:
      - "7077:7077"
    volumes:
      - ./data:/app/coze-discord-proxy/data
    environment:
      - USER_AUTHORIZATION=MTIwMjk2OTg4NTk3NDcyODcxNw.G5ZPp8.Bflh84OaP6RGdayDrIdJx60njqkWxLPToGhXqg,MTIxODQzMDg0NDA1NDY3MTM3MQ.GNEsBT.uJBCeTtjGonbxUhTuoGu3C0MuXBICy_Zr5r5rU
      - BOT_TOKEN=MTIxNzgyMTgzMDc0MDU3ODM5NQ.GyCltX.OkTnna6WWTjg-u0m_AcqYszKPxWhT0ZqWhQDSs
      - GUILD_ID=1202972464657858601
      - CHANNEL_ID=1202972464657858603
      - COZE_BOT_ID=1217821295866286170
      - PROXY_SECRET=sk-AGyYUpud*PMxnrQU
      - SWAGGER_ENABLE=0
      - TZ=Asia/Shanghai

```



//      - COZE_BOT_ID=1217821295866286170



```
[
  {
    "proxySecret": "sk-AGyYUpud*PMxnrQU",
    "cozeBotId": "1217821295866286170",
    "model": ["gpt-4"],
    "channelId": "1202972464657858603"
  },
  {
    "proxySecret": "sk-AGyYUpud*PMxnrQU",
    "cozeBotId": "1218435135985156187",
    "model": ["gpt-3.5"],
    "channelId": "1202972464657858603"
  }
]
```



```

```



## clash

秘钥

```
AGyYUpud*PMxnrQU
```

socket

```
AGyYUpud*PMxnrQU
```





```
请使用 http://8.210.2.174:9999/ui 管理内置规则
-----------------------------------------------
其他设备可以使用PAC配置连接：http://8.210.2.174:9999/ui/pac
或者使用HTTP/SOCK5方式连接：IP{8.210.2.174}端口{6666}

```



本地面板

```
http://8.210.2.174:9999/ui/#/connections
```

http://clash.razord.top/#/logs





## lobe

```
docker run -d -p 3210:3210 \
    -e OPENAI_API_KEY= \
    -e ACCESS_CODE=985641 \
    --name lobe-chat \
    lobehub/lobe-chat
```

