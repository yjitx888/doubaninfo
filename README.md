# DoubanInfo 豆影 API 接口

<img src="https://doubaninfo.com/favicon.svg" alt="DoubanInfo Logo" width="120" />

**DoubanInfo API** 是一款专业的豆瓣/IMDb电影结构化元数据抓取及排版服务。不仅支持极速获取高精度的影视结构数据（标准 JSON 格式），还独家内置了面向主流 PT 站点的 **无缝 BBCode 图文排版输出功能**。免去开发者二次爬虫洗数据排版的烦恼，为您打造私人及公共影音中枢的黄金搭档。

🔗 **官方文档地址**：https://doubaninfo.com/api/api_usage.php  
📌 **申请 Key 地址**：https://doubaninfo.com/user/dashboard.php

---

## ✨ 核心优势 & 适用场景

- 🚀 **极速双态输出 (JSON/BBCode)**：不仅提供结构化极强的 JSON 数据供程序解析，更独家提供 `&format` 参数直接吐出精美的 BBCode 纯文本，免去二次排版烦恼。
- 🧩 **深度结构化的元数据**：演职员（带头像/饰角）、类型、别名、多国语言及获奖记录等核心字段，全部经过严格清洗并封装为标准的 JSON Array，完美契合数据库入库。
- 🔗 **双边评分智能穿透**：豆瓣结果自动附带 IMDb 评分与人数；IMDb 结果亦精准对应各项基础数据。一次请求，抹平两大影库鸿沟。
- 🛡️ **高级风控盾与配额追踪**：底层异步集群抗高并发。JSON 响应自带 `usage` 字段，实时返回分钟级/天级调用配额消耗，业务调度清晰可控。
- 🔄 **灵活的双向转换映射**：内置智能源站映射引擎。输入豆瓣链接配合 `&imdb` 即可穿透拿 IMDb 数据，反之亦然。

**🎯 典型应用场景：**
* **PT/BT 论坛发种自动化**：使用 `&format` 直接获取 BBCode，完美无脑适配 NexusPHP 的 PTGEN 插件。
* **私人影音库 (Emby/Plex) 刮削器**：利用详尽的 `cast` 与 `director` 数组构建高质量中文元数据插件。
* **TG/Discord 查影机器人**：结合剧情简介与评分，实现秒回精美电影卡片。

---

## 📡 接口快速接入

- **基础 Endpoint**: `https://doubaninfo.com/api/v1_douban.php`
- **请求方法**: `GET`
- **鉴权建议**：支持 URL 参数传 `key=xxx`，但**更推荐在 HTTP Header 中通过 `X-API-KEY: xxx` 传递**以提升安全性。

### 请求参数

| 参数名称 | 必填 | 描述说明 |
| :------- | :--- | :--------------------------------------------------------------------------------------------------------------------------------- |
| `key`    | **是** | 您的 API 请求密钥（请在官网用户中心获取）。也可通过 Header `X-API-KEY` 传入。 |
| `url`    | **是** | **检索源**。支持精准传入 **豆瓣 ID/链接** 或 **IMDb ID**；最新版**更支持直接传入中英文影视名称模糊搜索**（如 `碟中谍 2025` 或 `Mission.Impossible`）。建议附加年份以提升精度。 |
| `format` | 否   | 可选值为 `bbcode` 或留空。追加 `&format` 则直接返回纯文本排版数据（适合 PT 发帖）；缺省则返回标准 JSON 结构。 |
| `douban` | 否   | 强制映射。提供 IMDb ID 但希望强制获取**对应的豆瓣影视信息**时使用，追加 `&douban=1`。 |
| `imdb`   | 否   | 强制映射。提供豆瓣 ID 但希望强制获取**对应的原始 IMDb 信息**时使用，追加 `&imdb=1`。 |
| `nocache`| 否   | <span style="color:orange">[VIP/SVIP 专属]</span> 强制跳过本地缓存，实时拉取上游最新数据。追加 `&nocache=1`。 |
| `sid`    | 否   | 兼容老版本的条目 ID 参数，功能等同于 `url`，建议废弃并统一迁移至 `url`。 |

---

## 🚦 HTTP 状态码说明与频控策略

为保证接口稳定性，系统配置了智能排队与防并发机制。请在业务层做好合理的延时重试（Retry）封装。

| 状态码 | 含义处理建议 |
| :------- | :--------------------------------------------------------------------------- |
| **200 OK** | 成功返回目标数据体。 |
| **400 Bad Request** | 参数错误。通常是因为缺少必填的 `url` 或 `key` 参数。 |
| **403 Forbidden** | 鉴权失败。API Key 无效、配额已用完，或您的账户已被封禁。 |
| **429 Too Many** | 触发频控。并发过高或额度达限，服务器拦截降级，请稍后等自然分钟刷新再试。 |
| **500/502** | 极端验证码拦截、内部服务链路抖动或外网源站拥塞。 |

---

## 💻 主流开发语言调用范例

### 🔹 1. cURL (Header 安全鉴权 - 获取 JSON)

```bash
curl -G "[https://doubaninfo.com/api/v1_douban.php](https://doubaninfo.com/api/v1_douban.php)" \
  -H "X-API-KEY: YOUR_API_KEY" \
  --data-urlencode "url=tt9603208"
```

### 🔹 2. Python (获取 BBCode 纯文本)

```python
import requests

url = "[https://doubaninfo.com/api/v1_douban.php](https://doubaninfo.com/api/v1_douban.php)"
params = {
    "url": "碟中谍 2025",
    "key": "YOUR_API_KEY",
    "format": ""      # 关键参数，有它则直接返回纯字符串
}

resp = requests.get(url, params=params, timeout=20)
print(resp.text)
```

### 🔹 3. PHP

```php
$url = "[https://doubaninfo.com/api/v1_douban.php](https://doubaninfo.com/api/v1_douban.php)?" . http_build_query([
    'url' => 'tt9603208',
    'key' => 'YOUR_API_KEY'
]);

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response, true);
print_r($data);
```

### 🔹 4. JavaScript (Node.js / Fetch API)

```javascript
const params = new URLSearchParams({
  url: '30433456',
  key: 'YOUR_API_KEY'
});

fetch(`https://doubaninfo.com/api/v1_douban.php?${params}`)
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## 📝 JSON 返回字段字典

以下涵盖了豆瓣与 IMDb 接口可能返回的核心 JSON 字段。*(注意：部分非关键字段在源站缺失时可能返回空字符串 `""` 或空数组 `[]`)*

### 系统与基础状态字段 (通用)
| 字段名 | 类型 | 说明 |
| :--- | :--- | :--- |
| `success` | Boolean | 解析是否成功 (`true` / `false`)。 |
| `site` | String | 数据来源标识，`"douban"` 或 `"imdb"`。 |
| `format` | String | 仅当传了 `&format` 时存在，包含完整的 BBCode 文本排版。 |
| `usage` | Object | 包含 `limits` (总额度) 和 `used` (已用)，显示分钟与天级配额。 |

### 影视核心元数据 (豆瓣/IMDb 聚合)
| 字段名 | 类型 | 说明 |
| :--- | :--- | :--- |
| `title` | String | 完整条目标题。豆瓣常包含中英文，IMDb为原声标题。 |
| `chinese_title` | String | **[仅豆瓣]** 智能剥离出的纯中文标题。 |
| `year` | String/Int | 影片发行或首播的年份。 |
| `summary` / `plot` | String | 剧情简介详情。 |
| `poster` / `image`| String | 高清电影海报直链地址（已转存 CDN）。 |
| `genre` / `genres`| Array | 影片类型数组（如 `["动作", "惊悚"]`）。 |
| `region` | String/Arr | 制片国家/地区。 |
| `duration`/`runtime`| String | 影片时长（如 `"170分钟"` 或 `"2h 49m"`）。 |
| `aka` | Array | 影片又名/别名，极适合用于搜索容错与刮削匹配。 |
| `imdb_id` | String | 绑定的 IMDb 唯一 ID (如 `tt9603208`)。 |
| `douban_rating` | String | 豆瓣平均分。 |
| `imdb_rating` | String | IMDb 平均分。 |

### 演职员阵容 (Cast & Crew)
| 字段名 | 类型 | 说明 |
| :--- | :--- | :--- |
| `director` | Array | 导演列表。 |
| `writer` | Array | 编剧列表。 |
| `cast` | Array | 主演及配音演员。可能包含带 `name`, `image`, `character`（饰角）的对象。 |
| `full_celebrities` | Object | **[仅豆瓣]** 全网最全的演职员字典。按职能（如 `导演 Director`）分类，内部包含详尽的艺人专属连接、角色分离名 (`character`) 及历史代表作 (`works`)。 |

---

## 附：JSON 返回示例缩略版

**豆瓣接口 JSON 返回结构：**

```json
{
  "success": true,
  "site": "douban",
  "sid": "30433456",
  "title": "碟中谍8：最终清算 Mission: Impossible - The Final Reckoning",
  "chinese_title": "碟中谍8：最终清算",
  "year": "2025",
  "director": [
    "韦德·伊斯特伍德 Wade Eastwood"
  ],
  "cast": [
    "汤姆·克鲁斯 Tom Cruise (饰 伊森·亨特 Ethan Hunt)",
    "海莉·阿特维尔 Hayley Atwell (饰 格蕾丝 Grace)",
    "文·瑞姆斯 Ving Rhames (饰 卢瑟 Luther Stickell)",
    "..."
  ],
  "genre": [
    "动作",
    "惊悚",
    "冒险"
  ],
  "imdb_id": "tt9603208",
  "release_date": "2025-05-30(中国大陆) / 2025-05-23(美国)",
  "duration": "170分钟",
  "douban_rating": "7.6",
  "imdb_rating": "7.2",
  "summary": "超级人工智能“智体”即将引爆全球核弹危机...",
  "full_celebrities": {
    "导演 Director": [
      {
        "name": "克里斯托弗·麦奎里 Christopher McQuarrie",
        "link": "[https://www.douban.com/personage/27504593/](https://www.douban.com/personage/27504593/)",
        "image": "[https://img1.doubanio.com/view/celebrity/m/public/p1535912054.09.jpg](https://img1.doubanio.com/view/celebrity/m/public/p1535912054.09.jpg)",
        "role": "导演 Director",
        "character": "",
        "works": ["碟中谍6：全面瓦解", "明日边缘"]
      }
    ],
    "演员 Cast": [
      {
        "name": "汤姆·克鲁斯 Tom Cruise",
        "image": "[https://img9.doubanio.com/](https://img9.doubanio.com/)...",
        "role": "演员 Actor (饰 伊森·亨特 Ethan Hunt)",
        "character": "(饰 伊森·亨特 Ethan Hunt)",
        "works": ["碟中谍6", "雨人"]
      }
    ]
  },
  "tier": "svip",
  "usage": {
    "limits": { "key_per_min": 300, "key_per_day": 100000 },
    "used": { "key_min": 1, "key_day": 3 }
  }
}
```

---

## 📜 声明与开源协议

版权所有 &copy; DoubanInfo Team。
使用即代表您已同意平台的 服务条款 及 隐私政策。请自觉履行良好的请求礼仪，严禁恶意刷量消耗系统资源。
