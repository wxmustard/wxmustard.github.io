# 对象存储数据库设计

bucket信息表（s3_buckets）

|   字段名    |   类型   | 长度 | 不能为空 |       备注       |
| :---------: | :------: | :--: | :------: | :--------------: |
|     id      |   自增   |      |    是    |     表自增id     |
| bucket_name | varchar  |  32  |    是    |   bucket的名称   |
|  create_id  |   char   |  8   |    是    |     创建者id     |
|  create_at  | datetime |      |    是    | bucket的创建时间 |

用户最高权限表（s3_users）

|    字段名     |   类型   | 长度 | 不能为空 |       备注       |
| :-----------: | :------: | :--: | :------: | :--------------: |
|      id       |   自增   |      |    是    |     表自增id     |
|  access_key   | varchar  |  64  |    是    |    access_key    |
| access_secret | varchar  |  64  |    是    |  access_secret   |
|   create_id   |   char   |  8   |    是    |     创建者id     |
|   create_at   | datetime |      |    是    |     创建时间     |
|   update_at   | datetime |      |          | 最近一次更新时间 |

bucket共享权限表（s3_shares）

|    字段名     |   类型   | 长度 | 不能为空 |                    备注                     |
| :-----------: | :------: | :--: | :------: | :-----------------------------------------: |
|      id       |   自增   |  8   |    是    |                  表自增id                   |
|  bucket_name  | varchar  |  32  |    是    |                bucket的名称                 |
|  access_key   | varchar  |  64  |    是    |                 access_key                  |
| access_secret | varchar  |  64  |    是    |                access_secret                |
| authorization |   enum   |  2   |    是    | 1:get权限 2:post权限 3:put权限 4:delete权限 |
| is_ infinity  |   bool   |      |    是    |            1:无穷  0: 有限(默认)            |
|   deadline    | datetime |      |          |                共享截止时间                 |
|  is_project   |   bool   |      |    是    |            1: 项目 0: 个人(默认)            |
|   share_id    |   char   |  8   |          |                被共享用户id                 |
|   create_id   |   char   |  8   |    是    |                  创建者id                   |
|   create_at   | datetime |      |    是    |                  创建时间                   |
|   update_id   |   char   |  8   |          |              最近一次更新者id               |
|   update_at   | datetime |      |          |              最近一次更新时间               |

> 创建者id指的是创建这条记录的用户id
>
> 更新者id指的是跟新这条记录的用户id