# 对象存储数据库设计

bucket信息表（s3_bucket）

|   字段名    |   类型   | 长度 | 不能为空 |       备注       |
| :---------: | :------: | :--: | :------: | :--------------: |
|     id      |   自增   |  8   |    是    |     表自增id     |
|  bucket_id  |   char   |  8   |    是    | bucket的唯一标识 |
| bucket_name | varchar  |  32  |    是    |   bucket的名称   |
|  owner_id   | varchar  |  8   |    是    |     所有者id     |
| bucket_time | datatime |      |    是    | bucket的创建日期 |

用户信息表（s3_user）

|   字段名    |  类型   | 长度 | 不能为空 |      备注      |
| :---------: | :-----: | :--: | :------: | :------------: |
|     id      |  自增   |  8   |    是    |    表自增id    |
|   user_id   | varchar |  8   |    是    | 用户的唯一标识 |
| user_access | varchar |  32  |    是    |   用户access   |
| user_secret | carchar |  32  |    是    |   用户secret   |

bucket共享权限表（s3_share）

|    字段名    |   类型   | 长度 | 不能为空 |                 备注                  |
| :----------: | :------: | :--: | :------: | :-----------------------------------: |
|      id      |   自增   |  8   |    是    |               表自增id                |
|  bucket_id   |   char   |  8   |    是    |           bucket的唯一标识            |
|   share_id   |   char   |  8   |    是    |             共享的用户id              |
|  share_time  | datatime |      |    是    |    bucket所有者共享该bucket的时间     |
| auth_gettime | datatime |      |    是    |     共享用户拥有get权限的截止时间     |
|   auth_get   |   int    |  2   |    是    | 0:用户未拥有get权限 1:用户拥有get权限 |
| auth_puttime | datatime |      |    是    |     共享用户拥有put权限的截止时间     |
|   auth_put   |   int    |  2   |    是    | 0:用户未拥有put权限 1:用户拥有put权限 |

