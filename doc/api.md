# HelloGitHub 接口文档

## 一、基本

**网址：https://hellogithub.com**

### 1.1 响应约定
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

### 1.2 认证

#### 1.2.1 Basic Auth 
- `username`：随便（可以不写）
- `password`：为 token（用于认证权限）

示例：
```
➜  ~ curl -u 随便:token  https://hellogithub.com/api/v1/
{
  "message": "HG API-v1 hello world."
}
```

#### 1.2.2 Header Auth
- `token`：为 token（用于认证权限）

示例：
```
➜  ~ curl -H 'X-HG-TOKEN: token'  https://hellogithub.com/api/v1/
{
  "message": "HG API-v1 hello world."
}
```

## 二、基础接口

### 2.1 获取所有分类信息
PATH：`GET /api/v1/category/`

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

### 2.2 获取所有期的信息
PATH：`GET /api/v1/volume/`

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

## 三、统计展示接口

统计展示接口时间范围字段：
- 开始时间：start_time
- 结束时间：end_time

时间字段的规则：开始时间和结束时间间隔最小为 1 天，最大为一个月（31天）。如果开始时间和结束时间相差为 1 天，聚合维度为**小时**；如果相差大于 1 天，聚合维度为**天**。

比如：要请求 2019年11月16日 的数据，则 start_time 为 2019年11月16日00:00:00，end_time 为 2019年11月17日00:00:00

**Tips：** 
- 默认返回昨天的数据，时间戳单位为秒。  
- 如果两者时间间隔超过 31 天，以开始时间为准，结束时间为开始时间往后推 31 天。

### 3.1 获取首页汇总展示数据
该接口用于统计首页汇总展示所有图表的概括图。

- 统计来源为饼图（from）
- 推荐项目点击为双折线图（click）
- 某一期月刊为条形图（volume）
- 公告栏点击数为双折线图（notice）

PATH：`GET /api/v1/traffic/views/`

参数：

| 名称     | 必须  | 类型 | 描述 |
| ------- | ----- | ----- | ----- |
| event   |   是  | string | 请求事件名（可选 from，click，volume，notice）|
| start_time | 否 | int    | 开始时间戳 |
| end_time   | 否 | int    | 结束时间戳 |

响应：
**Tips：** 每次请求返回一种 event 的数据，提高数据返回效率。

```
{
  "message": "OK",
  "payload": {
    "start_time": 1573713003,
    "end_time": 1573723003,
    
    #统计来源的数据(event=from)
    "view_data": {
      "all_count": 400,  #该时间段的总数
      "data": [
        {
          "id": 13,  #来源分类的id
          "referrer": "谷歌",  #来源
          "count": 30,  #数量
          "percent": 0.41  #占比
        },
        ...
      ]
    },

    #推荐项目点击的数据(event=click)
    "view_data": {
      "all_count": 300,
      "all_ip_count": 222,
      "per": "hour",  #时间聚合的维度分：day和hour
      "data": [
        {
          "timestamp": 1573713003, #时间戳
          "count": 440,  #某一时间段项目的点击数量
          "ip_count": 322 # IP数量
        },
        ...
      ]
    },

    #某一期月刊的数据（只显示最近一期和选择的时间无关， event=volume）
    "view_data": {
      "all_count": 100,
      "all_ip_count": 22,
      "volume_id": 44,  #月刊的id
      "volume_name": "41",  #月刊的名称
      "data": [
        {
          "category_name": "Go 项目",  #月刊中项目的分类
          "category_id": 7,  #分类id
          "count": 30,
        },
        ...
      ]
    },
    
    #公告栏点击的数据(event=notice)
    "view_data": {
      "all_count": 300,
      "all_ip_count": 222,
      "per": "hour",  #时间聚合的维度分：day和hour
      "data": [
        {
          "timestamp": 1573713003, #时间戳
          "count": 440,  #某一时间段公告栏的点击数量
          "ip_count": 322 # IP数量
        },
        ...
      ]
    }
  }
}
```

### 3.2 获取用户访问来源统计数据

用户访问来源统计数据用于图表展示。（折线图）

PATH：`GET /api/v1/traffic/from/view/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| start_time | 否  | int  | 开始时间戳 |
| end_time   | 否  | int  | 结束时间戳 |

响应：
```
{
  "message": "OK",
  "payload": {
    "start_time": 1573713003,
    "end_time": 1573723003,
    "per": "hour",  #时间聚合的维度分：day和hour
    "all_count": 400,  #该时间段的总数
    "data": [
      {
        "id": 13,  #来源分类的id
        "referrer": "谷歌",  #来源
        "count": 30,  #数量
        "percent": 0.41  #占比
        "timeline": [
          {
            "timestamp": 1573713003,
            "count": 3
          },
          ...
        ]
      },
      ...
      }
    ]
  }
}
```

### 3.3 获取推荐项目点击统计数据

推荐项目点击统计数据用于图表展示。（折线图）

PATH：`GET /api/v1/traffic/click/view/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| start_time | 否  | int  | 开始时间戳 |
| end_time   | 否  | int  | 结束时间戳 |

响应：
```
{
  "message": "OK",
  "payload": {
    "start_time": 1573713003,
    "end_time": 1573723003,
    "per": "hour",  #时间聚合的维度分：day和hour
    "all_count": 400,  #该时间段的总数
    "all_ip_count": 322,  #IP总数
    "data": [
        {
          "category_id": 1,
          "category_name": "Python 项目",
          "timestamp": "1573713003", #时间
          "count": 440,  #某一时间段项目的点击数量
          "ip_count": 322 # IP数量
        },
        ...
      ]
      }
    ]
  }
}
```

### 3.4 获取推荐项目点击详细数据

返回推荐项目点击统计列表的数据

PATH：`GET /api/v1/traffic/click/detail/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| page    | 否    | int    | 页数默认为第一页 |
| order   | 否    | string | 排序列（默认 stars，可选 count，ip_count） |
| asc     | 否    | int    | 默认为 0 降序，1 为升序 |
| start_time | 否 | int    | 开始时间戳 |
| end_time   | 否 | int    | 结束时间戳 |

响应：
```
{
  "message": "OK",
  "payload": {
    "start_time": 1573713003,
    "end_time": 1573723003,
    "per": "hour",  #时间聚合的维度分：day和hour
    "all_count": 400,
    "all_ip_count": 200,
    "current_page": 1,  #当前页码
    "page_count": 10,  #总页数
    "order": "stars",
    "asc": 0,
    "data": [
      {
        "repo_id": 13,  #项目唯一id
        "category_name": "Go 项目",
        "category_id": 7,
        "volume_id": 1,
        "volume_name": "01",  #月刊期数
        "name": "HelloGitHub",  #项目名称
        "stars": 3000,  #项目stars数
        "lang": "Python",  #项目主语言
        "url": "https://github.com/521xueweihan/HelloGitHub",  #项目地址
        "count": 30,  #数量
        "ip_count": 23  #IP数
      },
      ...
    ]
  }
}
```

### 3.5 获取某一期月刊的统计数据

某一期月刊的统计数据用于图表展示。（双条形图）

PATH：`GET /api/v1/traffic/periodical/view/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| volume_id   | 否 | int | 期刊的 id，默认为最新一期 |
| category_id | 否 | int | 期刊分类的 id，默认返回所有分类的数据|

响应：
```
{
  "message": "OK",
  "payload": {
    "all_count": 400,  #该时间段的总数
    "all_ip_count": 322,  #IP总数
    "data": [
        {
          "category_id": 1,
          "category_name": "Python 项目",
          "volume_id": 1,
          "volume_name": "01",
          "count": 440,  #某一时间段项目的点击数量
          "ip_count": 322 # IP数量
        },
        ...
      ]
      }
    ]
  }
}
```

### 3.6 获取某一期月刊的详细数据

某一期月刊的统计数据用于列表的数据。

PATH：`GET /api/v1/traffic/periodical/detail/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| volume_id | 否  | int    | 期刊的 id，默认为最新一期 |
| page     | 否   | int    | 页数默认为第一页 |
| order    | 否   | string | 排序列（默认 stars，可选 count，ip_count） |
| asc      | 否   | int    | 默认为 0 降序，1 为升序 |


响应：
```
{
  "message": "OK",
  "payload": {
    "volume_id": 1,
    "volume_name": "01",  #月刊期数
    "all_count": 400,
    "all_ip_count": 200,
    "current_page": 1,  #当前页码
    "page_count": 10,  #总页数
    "order": "stars",
    "asc": 1,
    "data": [
      {
        "repo_id": 13,  #项目唯一id
        "category_name": "Go 项目",
        "category_id": 7,
        "name": "HelloGitHub",  #项目名称
        "stars": 3000,  #项目stars数
        "lang": "Python",  #项目主语言
        "url": "https://github.com/521xueweihan/HelloGitHub",  #项目地址
        "count": 30,  #数量
        "ip_count": 23  #IP数
      },
      ...
    ]
  }
}
```


## 四、日报汇总展示接口
以下 start_time 和 end_time 参考**第三部分**的相关描述。

### 4.1 获取语言分类

PATH：`GET /api/v1/daily/langs/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| start_time | 否 | int   | 开始时间戳 |
| end_time   | 否 | int   | 结束时间戳 |

响应：
```
{
  "message": "OK",
  "payload": {
    "start_time": 1573713003,
    "end_time": 1573723003,
    "langs": []  # 所有语言分类       
  }
}
```

### 4.2 获取 GitHub Source 上收集项目的详细数据

PATH：`GET /api/v1/daily/report/`

参数：

| 名称     | 必须  | 类型 | 描述 | 
| ------- | ----- | ----- | ----- |
| start_time | 否 | int    | 开始时间戳 |
| end_time   | 否 | int    | 结束时间戳 |
| lang       | 否 | string | 语言 |
| page       | 否 | int    | 页数默认为第一页 |
| order      | 否 | string | 排序列（默认 stars，可选 forks，repo_pushed_time） |
| asc        | 否 | int    | 默认为 0 降序，1 为升序 |

响应：
```
{
  "message": "OK",
  "payload": {
    "start_time": 1573713003,
    "end_time": 1573723003,
    "primary_lang": "",  #默认为空,即不区分语言
    "current_page": 1,  #当前页码
    "page_count": 10,  #总页数
    "order": "stars",
    "asc": 0,
    "count": 10,  #项目总数
    "page_size": 10,
    "data": [
      {
        "description": "Consul is a distributed, highly available, and data center aware solution to connect and configure applications across dynamic, distributed infrastructure.", 
        "is_chinese": false, 
        "name": "consul", 
        "primary_lang": "Go", 
        "repo_pushed_time": 1592275530, # 项目最后一次活动时间
        "stars": 19431, 
        "forks": 332, 
        "url": "https://github.com/hashicorp/consul"
      },
      ...
    ]
  }
}
```