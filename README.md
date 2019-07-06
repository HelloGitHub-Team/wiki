# HG 公开接口文档
## 一、基本
### 1.1 请求约定
**域名：https://hellogithub.com**

- GET 方法，参数只有一个 `q`。所有的参数 json 序列化后，通过 `q` 参数传递，具体参数说明下面的接口说明会说明
- POST 方法，参数全部通过 json 提交。`hearder` key-value 对为：`Content-Type:application/json`

例如：
- 获取月刊第 1 期的内容：
  ```
  https://hellogithub.com/api/v1/volume/projects/?q={"id": 1}
  ```
- 获取 C# 分类的内容：
  ```
  http://hellogithub.com//api/v1/category/projects/?q={"id": 9, "page":1}
  ```


#### 1.2 响应约定
响应状态通过 HTTP status code 反应，规则如下：
- 200：成功
- 400：参数错误
- 401：未登录
- 403：禁止访问
- 404：未找到
- 500：服务器异常

成功响应模版：
```
{
  "message": "OK", # 成功一般为 OK
  "payload": [] # 请求返回的数据
}
```

**Tips：** 字段说明会在响应例子中以 `#` 出现

#### 1.3 认证
采用 Basic Auth 方式：
- `username`：随便（可以不写）
- `password`：为 token（用于认证权限）

示例：
```
➜  ~ curl -u 随便:token  https://hellogithub.com/api/v1/
{
  "message": "HG API-v1 hello world."
}
```

## 二、获取类接口说明
### 2.1 按照期数获取项目
PATH：`/api/v1/volume/projects/`

方法：`GET`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| id    | 否    | int | 某一期的id |

**Tips：** 默认返回最新一期


响应：
```
{
    "message": "OK",
    "payload": [
        {
            "category": "其它", # 项目类别
            "description": "（英文）这个项目收集了很多 SEO 优化的建议\n", # 描述（Markdown语法）
            "sort_desc": "谷歌 Material Design 设计风格控件库", # 简短描述，纯文本
            "img_url": "", # 配图地址
            "name": "search-engine-optimization", # 项目名称
            "update_time": "2018-11-28 13:56:00", # 发布时间
            "url": "https://github.com/marcobiedermann/search-engine-optimization", # 项目地址
            "volume": "32" # 第几期
        },
        ....
        ]
}

```

### 2.2 按照分类获取项目
PATH：`/api/v1/category/projects/`

方法：`GET`

参数：

| 名称     | 必须  | 类型 | 描述 |
| ------- | ----- | ----- | ----- |
| id      | 否 | int | 某一类别的id |
| page    | 否 | int | 第几页（1为第一页） |

**Tips：** 默认随机返回一个类别的第一页，每页10个项目

响应：
```
{
    "current_page": , # 当前页
    "has_more": true, # 是否下一页还有数据
    "message": "OK",
    "payload": [
        {
            "category": "Python 项目",
            "description": "使用该库可以优雅地实现各种需求的重试。示例代码如下",
            "sort_desc": "谷歌 Material Design 设计风格控件库", # 简短描述，纯文本
            "img_url": "",
            "name": "tenacity",
            "update_time": "2018-06-27 22:26:19",
            "url": "https://github.com/jd/tenacity",
            "volume": "27"
        },
        ...
        ]
}
```

### 2.3 获取所有分类信息
PATH：`/api/v1/category/`

方法：`GET`

参数：无

响应：
```
{
    "message": "OK",
    "payload": [
        {
            "id": 11,
            "name": "C 项目"
        },
        {
            "id": 9,
            "name": "C# 项目"
        },
        ...
        ]
}
```

### 2.4 获取所有期的信息
PATH：`/api/v1/volume/`

方法：`GET`

参数：无

响应：
```
{
    "message": "OK",
    "payload": [
        {
            "id": 34,
            "name": "31",
            "update_time": "2018-09-28 10:31:25"
        },
        ...
        ]
}
```
## 三、推荐项目接口
PATH：`/api/v1/project/recommend/`

方法：`POST`

参数：

| 名称     | 必须 | 类型 | 描述 |
| ------- | ----- | ----- | ----- |
| title  |  是 | string | 标题 |
| description |  是 | string | 对该项目的描述 |
| url    |  是 | string  | 推荐项目的地址（最长150）|
| category  |  否 | string  | 推荐项目的类别 |
| langPrimary  |  是 | string  | 推荐项目的主要编程语言 |
| watch  |  否 | int  | 推荐项目的 watch 数量 |
| star  |  否 | int  | 推荐项目的 star 数量 |
| fork  |  否 | int  | 推荐项目的 fork 数量 |
| source   |  是 | string  | 用于标记提交的来源 |
| device   |  是 | string  | 用于标记提交的设备 |


**Tips：** 根据 url 校验是否重复，如果重复记录第一次推荐的内容。就算推荐的项目重复也返回 `200 OK`

响应：
```
{
    "message": "OK"
}
```
