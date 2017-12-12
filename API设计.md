- ​
- 请求方式

| http verbs | 描述   |
| ---------- | ---- |
| get        | 取出资源 |
| post       | 新建资源 |
| put        | 更新资源 |
| delete     | 删除资源 |

- url

| url                                      | 描述              |
| ---------------------------------------- | --------------- |
| get http:// 127.0.0.1:8000/v1/vps/       | 返回虚拟机列表         |
| get http:// 127.0.0.1:8000/v1/vps/usrid/{usrid} | 返回usrid用户的虚拟机信息 |
| post http:// 127.0.0.1:8000/v1/vps/usrid/{usrid} | 用户新建虚拟机         |
| put http:// 127.0.0.1:8000/v1/vps/usrid/{usrid} | 用户更改虚拟机信息       |
| delete http:// 127.0.0.1:8000/v1/vps/usrid/{usrid} | 用户删除虚拟机         |
|                                          |                 |
|                                          |                 |
|                                          |                 |
|                                          |                 |

```
https://cvm.api.qcloud.com/v2/index.php?Action=DescribeInstanceTypeConfigs
https://cvm.api.qcloud.com/v2/instances/{instanceid}
```

- 参数
- 状态码