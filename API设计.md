- 请求方式

| http verbs | 描述   |
| ---------- | ---- |
| get        | 取出资源 |
| post       | 新建资源 |
| put        | 更新资源 |
| delete     | 删除资源 |

- url

| url                                      | 描述        |
| ---------------------------------------- | --------- |
| get http:// 127.0.0.1:8000/v1/vps/       | 返回虚拟机列表   |
| get http:// 127.0.0.1:8000/v1/vps/kvmid/{kvmid} | 返回虚拟机信息   |
| post http:// 127.0.0.1:8000/v1/vps/kvmid/{kvmid} | 用户新建虚拟机   |
| put http:// 127.0.0.1:8000/v1/vps/kvmid/{kvmid} | 用户更改虚拟机信息 |
| delete http:// 127.0.0.1:8000/v1/vps/kvmid/{kvmid} | 用户删除虚拟机   |
|                                          |           |
|                                          |           |
|                                          |           |
|                                          |           |

- 参数
- 状态码

