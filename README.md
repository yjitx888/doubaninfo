# DoubanInfo 豆影 API 接口

<img src="https://doubaninfo.com/favicon.svg" alt="DoubanInfo Logo" width="120" />

**DoubanInfo API** 是一款专业的豆瓣/IMDb电影结构化元数据抓取及排版服务。不仅支持极速获取高精度的影视结构数据（标准 JSON 格式），还独家内置了面向主流 PT 站点的 **无缝 BBCode 图文排版输出功能**。免去开发者二次爬虫洗数据排版的烦恼，为您打造私人及公共影音中枢的黄金搭档。

🔗 **官方文档地址**：[https://doubaninfo.com/api/api_usage.php](https://doubaninfo.com/api/api_usage.php)  
📌 **申请 Key 地址**：[https://doubaninfo.com/user/dashboard.php](https://doubaninfo.com/user/dashboard.php)

---

## ✨ 核心亮点

- 🚀 **极速双态响应**：通过追加 `&format` 参数，不仅提供给代码解析的 **JSON (Array)**，也提供人类直接可粘贴的精密 **BBCode（纯文本）**格式。
- 🧩 **全量元数据解析**：精细解析出含有头像与角色分工的阵容（Cast）、多语言地区、发售/首映年表及细致深入的获奖情况。
- 🛡️ **底层智能降级引擎**：以庞大的豆瓣源数据为基石，若遇缺漏将全自动补充 IMDb、TMDB、Bangumi 等正版库关联的补充信息与防盗图床高速海报。
- ⚡ **无死角的容错搜索**：全面改版的搜索核心，除完美的纯数字豆瓣 ID、[tt](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/util.php#132-158) 开头 IMDb ID 匹配外，更强力支持**原中文片名百分百匹配搜索**（例如直接传 `碟中谍` 或自带年份的 `碟中谍1996` 以及点号 `Mission.Impossible`）。

---

## 📡 接口快速接入

- **基础 Endpoint**: `https://doubaninfo.com/api/v1_douban.php`
- **请求方法**: `GET / POST`
- **内容类型**: `application/x-www-form-urlencoded;charset=UTF-8`

### 请求参数

| 字段名称 | 类型   | 必填 | 描述说明 |
| :------- | :----- | :--- | :--------------------------------------------------------------------------------------------------------------------------------- |
| [url](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/upstream.php#5-12)    | String | **是** | **检索源**。支持 豆瓣 ID（如 `30433456`），也可传入完整豆瓣链接；支持 IMDb ID（如 `tt0093593`）；更支持直接输入影视名称或中外文别名模糊精准匹配。 |
| [key](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/util.php#20-29)    | String | **是** | 您的鉴权私钥。新用户需在官网注册申领。 |
| `format` | String | 否   | （可选）。仅传入 `&format` 或 `format=bbcode` 时，系统将直接返回纯文本 BBCode 排版结构；缺省此字段则返回纯正 JSON 集。若搜索匹配了多条结果，BBCode 模式下将自动选返回第一条（通常为最新版），JSON 模式则返回完整的可供代码供选择的结果数组列表。 |
| `sid`    | String | 否   | 老版本向下兼容参数。功能等价于 [url](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/upstream.php#5-12)，建议直接迁移至 [url](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/upstream.php#5-12) 标准参数。 |

---

## 🚦 HTTP 状态码说明与频控策略

我们建立有免费、VIP及 SVIP 等不同频率限额的机制。API 后台采取 **严格的自然分钟级窗口（Fixed Window）防刷统计**。在每分钟额定请求限制内，允许秒级的高并发突发峰值，用光额度后该分钟剩余秒数报 429 排队。

| HTTP 状态码 | 含义处理建议 |
| :------------ | :--------------------------------------------------------------------------- |
| **200 OK** | 成功返回目标数据体。 |
| **400 Bad Request** | 未提供必填的 [url](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/upstream.php#5-12) 或 [key](file:///c:/Users/pterv/Desktop/fsdownload/doubaninfo.com/lib/util.php#20-29) 短信缺失。 |
| **403 Forbidden** | 后台风控提示由于私钥错误、非法 IP 请求或封号违规被阻断。 |
| **429 Too Many** | 并发峰值已满/额度达限，服务器拦截降级，请稍后等自然分刷新再请求一次。 |
| **500/502** | 极端验证码拦截、内部服务链路抖动或外网源站拥塞。需业务层代码做友好延时充试封装。 |

---

## 💻 主流开发语言调用范例

### 🔹 1. cURL (标准带 Key 返回 JSON)

```bash
curl -G "https://doubaninfo.com/api/v1_douban.php" \
  --data-urlencode "url=tt9603208" \
  --data-urlencode "key=YOUR_API_KEY"
```

### 🔹 2. Python (返回 BBCode 文本排版)

```python
import requests

url = "https://doubaninfo.com/api/v1_douban.php"
params = {
    "url": "碟中谍1996",  # 支持模糊中英文片名！
    "key": "YOUR_API_KEY",
    "format": ""      # 关键参数，有它则直接抛回纯字符串
}

response = requests.get(url, params=params)
print(response.text)
```

### 🔹 3. Node.js / JavaScript (Fetch API)

```javascript
const query = new URLSearchParams({
  url: '30433456',
  key: 'YOUR_API_KEY'
});

fetch(`https://doubaninfo.com/api/v1_douban.php?${query}`)
  .then(res => res.json())
  .then(jsonResult => {
      // 当发生多条名称搜索词冲突时，jsonResult 包含 'status' === 'multiple' 与 results 队列
      console.log(jsonResult); 
  })
  .catch(err => console.error(err));
```

### 🔹 4. PHP

```php
$params = [
    'url' => 'Mission.Impossible',
    'key' => 'YOUR_API_KEY'
];

$url = "https://doubaninfo.com/api/v1_douban.php?" . http_build_query($params);

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response, true);
print_r($data);
```

---

## 📝 完整 JSON 返回体范例参考

以下为搜索 `30433456` 的默认 JSON 返回范例（非 BBCode 模式）：

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
    "西蒙·佩吉 Simon Pegg (饰 班吉 Benji Dunn)",
    "埃塞·莫拉雷斯 Esai Morales (饰 盖布瑞尔 Gabriel)",
    "庞·克莱门捷夫 Pom Klementieff (饰 帕丽斯 Paris)",
    "亨利·科泽尼 Henry Czerny (饰 尤金·基特里奇 Eugene Kittridge)",
    "霍特·麦克卡兰尼 Holt McCallany (饰 塞林 Secretary of Defense Bernstein)",
    "珍妮特·麦克蒂尔 Janet McTeer (饰 沃尔特斯 Walters)",
    "尼克·奥弗曼 Nick Offerman (饰 西德尼 Sydney)",
    "汉娜·沃丁厄姆 Hannah Waddingham (饰 尼丽 Admiral Neely)",
    "特拉梅尔·提尔曼 Tramell Tillman (饰 布莱索 Captain Bledsoe)",
    "安吉拉·贝塞特 Angela Bassett (饰 艾莉卡·史隆 Erica Sloan)",
    "谢伊·惠格姆 Shea Whigham (饰 布里格斯 Jasper Briggs)",
    "格雷格·泰山·戴维斯 Greg Tarzan Davis (饰 德加 Degas)",
    "查尔斯·帕内尔 Charles Parnell (饰 理查兹 Richards)",
    "马克·加蒂斯 Mark Gatiss (饰 埃斯特朗 Angstrom)",
    "罗尔夫·萨克森 Rolf Saxon (饰 威廉·多诺利 William Donloe)",
    "露西·图卢加鲁克 Lucy Tulugarjuk (饰 塔蓓萨 Tapeesa)",
    "加利·艾尔维斯 Cary Elwes (饰 Denlinger)",
    "凯蒂·M·奥布莱恩 Katy M. O'Brian (饰 Kodiak)",
    "帕沙·D·林奇尼科夫 Pasha D. Lychnikoff (饰 Captain Koltsov)",
    "艾琳·巴特尔 Erin Battle (饰 Sergeant Armstrong)",
    "托米·厄尔·詹金斯 Tommie Earl Jenkins (饰 Colonel Burdick)",
    "迪娜·特鲁迪 Deena Trudy (饰 Agent)",
    "罗伯·德兰尼 Rob Delaney",
    "凯尔·弗里曼特尔 Kyle Freemantle (饰 Gabriel's Henchman)",
    "托马斯·帕雷德斯 Tomás Paredes (饰 Hagar (as Tomas Paredes)",
    "雷内·弗拉贝尔 Rene Vrabel (饰 Gabriel's Henchman)",
    "伊戈尔·卡尔波维奇 Igor Karpovich (饰 Gabriel's Henchman)"
  ],
  "genre": [
    "动作",
    "惊悚",
    "冒险"
  ],
  "imdb_id": "tt9603208",
  "writer": [
    "克里斯托弗·麦奎里 Christopher McQuarrie",
    "埃里克·延德雷森 Erik Jendresen",
    "布鲁斯·盖勒 Bruce Geller"
  ],
  "release_date": "2025-05-30(中国大陆) / 2025-05-14(戛纳国际电影节) / 2025-05-23(美国)",
  "premiere_date": "",
  "season_count": "",
  "region": "美国",
  "language": "英语 / 法语",
  "duration": "170分钟",
  "aka": [
    "Mission: Impossible - The Final Reckoning",
    "碟中谍8",
    "不可能的任务：最终清算(台)",
    "职业特工队：最终清算(港)",
    "碟中谍8：致命清算(下)",
    "Mission: Impossible – Dead Reckoning Part Two",
    "Mission: Impossible 8",
    "MI8"
  ],
  "runtime": "170分钟",
  "douban_rating": "7.6",
  "douban_rating_average": "7.6",
  "douban_votes": "268158",
  "summary": "超级人工智能“智体”即将引爆全球核弹危机，把世界推向毁灭边缘。而伊森·亨特（汤姆·克鲁斯 饰）和他的IMF小队在上次行动中遭遇重创，团队濒临分崩离析。虽然伊森已获得关闭“智体”的钥匙，但要彻底消灭“智体”，完成这一拯救全人类的终极任务，仍需要IMF小队齐心协力突破重重困难。他们不仅要面对全知全能又无影无形的“智体”与其手下盖布瑞尔（埃塞·莫拉雷斯 饰）的百般阻拦，还要解决来自过去的种种恩怨。每个抉择都关乎信念与命运，等待伊森和IMF小队的，究竟是怎样的结局？",
  "poster": "https://img2.pixhost.to/images/6519/705755077_30433456.jpg",
  "cover": "https://img2.pixhost.to/images/6519/705755077_30433456.jpg",
  "awards": [
    "第83届金球奖 (2026) 电影类 电影和票房成就奖(提名)",
    "第49届日本电影学院奖 (2026) 最佳外语片(提名)",
    "第32届美国演员奖 (2026) 电影奖 电影最佳特技群戏",
    "第31届美国评论家选择电影奖 (2026) 最佳视觉效果(提名) 亚历克斯·伍特克 / 克里斯汀·霍尔 / 杰夫·萨瑟兰 / 伊安·洛 / 最佳特技设计 韦德·伊斯特伍德"
  ],
  "full_celebrities": {
    "导演 Director": [
      {
        "name": "克里斯托弗·麦奎里 Christopher McQuarrie",
        "link": "https://www.douban.com/personage/27504593/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p1535912054.09.jpg",
        "role": "导演 Director",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "明日边缘",
          "碟中谍5：神秘国度"
        ]
      }
    ],
    "演员 Cast": [
      {
        "name": "汤姆·克鲁斯 Tom Cruise",
        "link": "https://www.douban.com/personage/27260203/",
        "image": "https://img9.doubanio.com/view/personage/m/public/341b1e42f0d9e68426c414744e7ebd34.jpg",
        "role": "演员 Actor (饰 伊森·亨特 Ethan Hunt)",
        "character": "(饰 伊森·亨特 Ethan Hunt)",
        "works": [
          "碟中谍6：全面瓦解",
          "雨人",
          "明日边缘"
        ]
      },
      {
        "name": "海莉·阿特维尔 Hayley Atwell",
        "link": "https://www.douban.com/personage/27205741/",
        "image": "https://img1.doubanio.com/view/personage/m/public/54c8baf2ace4f805cf4b39b4bf45493c.jpg",
        "role": "演员 Actress (饰 格蕾丝 Grace)",
        "character": "(饰 格蕾丝 Grace)",
        "works": [
          "复仇者联盟4：终局之战",
          "复仇者联盟2：奥创纪元",
          "美国队长2"
        ]
      },
      {
        "name": "文·瑞姆斯 Ving Rhames",
        "link": "https://www.douban.com/personage/27253890/",
        "image": "https://img1.doubanio.com/view/personage/m/public/edf36ca17d466bd7b4fd336005887239.jpg",
        "role": "演员 Actor (饰 卢瑟 Luther Stickell)",
        "character": "(饰 卢瑟 Luther Stickell)",
        "works": [
          "碟中谍6：全面瓦解",
          "低俗小说",
          "银河护卫队2"
        ]
      },
      {
        "name": "西蒙·佩吉 Simon Pegg",
        "link": "https://www.douban.com/personage/27241396/",
        "image": "https://img1.doubanio.com/view/personage/m/public/487db5b99e05e706829a581f900c03bd.jpg",
        "role": "演员 Actor (饰 班吉 Benji Dunn)",
        "character": "(饰 班吉 Benji Dunn)",
        "works": [
          "头号玩家",
          "碟中谍6：全面瓦解",
          "碟中谍4"
        ]
      },
      {
        "name": "埃塞·莫拉雷斯 Esai Morales",
        "link": "https://www.douban.com/personage/27233626/",
        "image": "https://img1.doubanio.com/view/personage/m/public/2557275827563b0db3f8bf122e4faccc.jpg",
        "role": "演员 Actor (饰 盖布瑞尔 Gabriel)",
        "character": "(饰 盖布瑞尔 Gabriel)",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算",
          "黑钱胜地"
        ]
      },
      {
        "name": "庞·克莱门捷夫 Pom Klementieff",
        "link": "https://www.douban.com/personage/27541230/",
        "image": "https://img3.doubanio.com/view/personage/m/public/d23a626882e4aa3dc52af5a000098462.jpg",
        "role": "演员 Actress (饰 帕丽斯 Paris)",
        "character": "(饰 帕丽斯 Paris)",
        "works": [
          "复仇者联盟3：无限战争",
          "复仇者联盟4：终局之战",
          "银河护卫队2"
        ]
      },
      {
        "name": "亨利·科泽尼 Henry Czerny",
        "link": "https://www.douban.com/personage/27212704/",
        "image": "https://img1.doubanio.com/view/personage/m/public/d3e7952a2c9fb753c079a73262b7dbfb.jpg",
        "role": "演员 Actor (饰 尤金·基特里奇 Eugene Kittridge)",
        "character": "(饰 尤金·基特里奇 Eugene Kittridge)",
        "works": [
          "碟中谍",
          "碟中谍7：致命清算",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "霍特·麦克卡兰尼 Holt McCallany",
        "link": "https://www.douban.com/personage/27323876/",
        "image": "https://img2.doubanio.com/view/personage/m/public/03f8a333fcda0202e9cc9be7908f8b61.jpg",
        "role": "演员 Actor (饰 塞林 Secretary of Defense Bernstein)",
        "character": "(饰 塞林 Secretary of Defense Bernstein)",
        "works": [
          "搏击俱乐部",
          "萨利机长",
          "正义联盟"
        ]
      },
      {
        "name": "珍妮特·麦克蒂尔 Janet McTeer",
        "link": "https://www.douban.com/personage/27250767/",
        "image": "https://img1.doubanio.com/view/personage/m/public/f26cba87638f8d5ba9163d65cb5be27d.jpg",
        "role": "演员 Actress (饰 沃尔特斯 Walters)",
        "character": "(饰 沃尔特斯 Walters)",
        "works": [
          "遇见你之前",
          "沉睡魔咒",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "尼克·奥弗曼 Nick Offerman",
        "link": "https://www.douban.com/personage/27215056/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p5870.jpg",
        "role": "演员 Actor (饰 西德尼 Sydney)",
        "character": "(饰 西德尼 Sydney)",
        "works": [
          "欢乐好声音",
          "精灵旅社2",
          "罪恶之城"
        ]
      },
      {
        "name": "汉娜·沃丁厄姆 Hannah Waddingham",
        "link": "https://www.douban.com/personage/27498349/",
        "image": "https://img1.doubanio.com/view/personage/m/public/0821890761d2a700366b79584e9afe1d.jpg",
        "role": "演员 Actress (饰 尼丽 Admiral Neely)",
        "character": "(饰 尼丽 Admiral Neely)",
        "works": [
          "悲惨世界",
          "性爱自修室",
          "权力的游戏"
        ]
      },
      {
        "name": "特拉梅尔·提尔曼 Tramell Tillman",
        "link": "https://www.douban.com/personage/35782850/",
        "image": "https://img1.doubanio.com/view/personage/m/public/95a2f3045383635631d824eb47afebfd.jpg",
        "role": "演员 Actor (饰 布莱索 Captain Bledsoe)",
        "character": "(饰 布莱索 Captain Bledsoe)",
        "works": [
          "人生切割术",
          "碟中谍8：最终清算",
          "人生切割术"
        ]
      },
      {
        "name": "安吉拉·贝塞特 Angela Bassett",
        "link": "https://www.douban.com/personage/27230946/",
        "image": "https://img1.doubanio.com/view/personage/m/public/2a750c65cce8893d6a6a4ca327baf7b0.jpg",
        "role": "演员 Actress (饰 艾莉卡·史隆 Erica Sloan)",
        "character": "(饰 艾莉卡·史隆 Erica Sloan)",
        "works": [
          "复仇者联盟4：终局之战",
          "碟中谍6：全面瓦解",
          "心灵奇旅"
        ]
      },
      {
        "name": "谢伊·惠格姆 Shea Whigham",
        "link": "https://www.douban.com/personage/27371069/",
        "image": "https://img3.doubanio.com/view/celebrity/m/public/p1575380120.57.jpg",
        "role": "演员 Actor (饰 布里格斯 Jasper Briggs)",
        "character": "(饰 布里格斯 Jasper Briggs)",
        "works": [
          "小丑",
          "华尔街之狼",
          "速度与激情6"
        ]
      },
      {
        "name": "格雷格·泰山·戴维斯 Greg Tarzan Davis",
        "link": "https://www.douban.com/personage/35925821/",
        "image": "https://img3.doubanio.com/view/personage/m/public/0864cd1c52e23395c61211d36031c527.jpg",
        "role": "演员 Actor (饰 德加 Degas)",
        "character": "(饰 德加 Degas)",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算",
          "壮志凌云2：独行侠"
        ]
      },
      {
        "name": "查尔斯·帕内尔 Charles Parnell",
        "link": "https://www.douban.com/personage/27554677/",
        "image": "https://img2.doubanio.com/view/celebrity/m/public/p1546676648.01.jpg",
        "role": "演员 Actor (饰 理查兹 Richards)",
        "character": "(饰 理查兹 Richards)",
        "works": [
          "变形金刚4：绝迹重生",
          "碟中谍7：致命清算",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "马克·加蒂斯 Mark Gatiss",
        "link": "https://www.douban.com/personage/27233710/",
        "image": "https://img3.doubanio.com/view/celebrity/m/public/p41033.jpg",
        "role": "演员 Actor (饰 埃斯特朗 Angstrom)",
        "character": "(饰 埃斯特朗 Angstrom)",
        "works": [
          "神探夏洛克",
          "困在时间里的父亲",
          "神探夏洛克"
        ]
      },
      {
        "name": "罗尔夫·萨克森 Rolf Saxon",
        "link": "https://www.douban.com/personage/27576132/",
        "image": "https://img2.doubanio.com/view/personage/m/public/732cb09c59dcb5847de9af242aea7531.jpg",
        "role": "演员 Actor (饰 威廉·多诺利 William Donloe)",
        "character": "(饰 威廉·多诺利 William Donloe)",
        "works": [
          "拯救大兵瑞恩",
          "碟中谍",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "露西·图卢加鲁克 Lucy Tulugarjuk",
        "link": "https://www.douban.com/personage/27490332/",
        "image": "https://img9.doubanio.com/view/personage/m/public/fe1d338d0823c16f6d8c45d5e7dcb345.jpg",
        "role": "演员 Actress (饰 塔蓓萨 Tapeesa)",
        "character": "(饰 塔蓓萨 Tapeesa)",
        "works": [
          "碟中谍8：最终清算",
          "冰原快跑人",
          "我们彼此凝望"
        ]
      },
      {
        "name": "加利·艾尔维斯 Cary Elwes",
        "link": "https://www.douban.com/personage/27246275/",
        "image": "https://img9.doubanio.com/view/personage/m/public/e98bf44e0859d554ac1cabb0b90cdbe6.jpg",
        "role": "演员 Actor (饰 Denlinger)",
        "character": "(饰 Denlinger)",
        "works": [
          "侧耳倾听",
          "电锯惊魂",
          "红猪"
        ]
      },
      {
        "name": "凯蒂·M·奥布莱恩 Katy M. O'Brian",
        "link": "https://www.douban.com/personage/30408244/",
        "image": "https://img1.doubanio.com/view/personage/m/public/adb38f85475c618e40b375c707e44a78.jpg",
        "role": "演员 Actress (饰 Kodiak)",
        "character": "(饰 Kodiak)",
        "works": [
          "碟中谍8：最终清算",
          "蚁人与黄蜂女：量子狂潮",
          "曼达洛人"
        ]
      },
      {
        "name": "帕沙·D·林奇尼科夫 Pasha D. Lychnikoff",
        "link": "https://www.douban.com/personage/27329380/",
        "image": "https://img3.doubanio.com/view/celebrity/m/public/p1352806312.13.jpg",
        "role": "演员 Actor/Actress (饰 Captain Koltsov)",
        "character": "(饰 Captain Koltsov)",
        "works": [
          "子弹列车",
          "碟中谍8：最终清算",
          "生活大爆炸"
        ]
      },
      {
        "name": "艾琳·巴特尔 Erin Battle",
        "link": "https://www.douban.com/personage/37038161/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "演员 Actor/Actress (饰 Sergeant Armstrong)",
        "character": "(饰 Sergeant Armstrong)",
        "works": [
          "碟中谍8：最终清算",
          "魔法坏女巫2"
        ]
      },
      {
        "name": "托米·厄尔·詹金斯 Tommie Earl Jenkins",
        "link": "https://www.douban.com/personage/34895883/",
        "image": "https://img3.doubanio.com/view/personage/m/public/ad5d08776ac5bb77be886b1022177802.jpg",
        "role": "演员 Actor (饰 Colonel Burdick)",
        "character": "(饰 Colonel Burdick)",
        "works": [
          "碟中谍8：最终清算",
          "星期三",
          "末日逃生2：迁移"
        ]
      },
      {
        "name": "迪娜·特鲁迪 Deena Trudy",
        "link": "https://www.douban.com/personage/30241274/",
        "image": "https://img2.doubanio.com/view/celebrity/m/public/p1543973338.21.jpg",
        "role": "演员 Actor/Actress (饰 Agent)",
        "character": "(饰 Agent)",
        "works": [
          "大黄蜂",
          "碟中谍8：最终清算",
          "机器人战警"
        ]
      },
      {
        "name": "罗伯·德兰尼 Rob Delaney",
        "link": "https://www.douban.com/personage/27474156/",
        "image": "https://img3.doubanio.com/view/personage/m/public/df78cd1c32374094508f0a5d9ed81063.jpg",
        "role": "演员 Actor",
        "character": "",
        "works": [
          "死侍2：我爱我家",
          "死侍与金刚狼",
          "碟中谍7：致命清算"
        ]
      },
      {
        "name": "凯尔·弗里曼特尔 Kyle Freemantle",
        "link": "https://www.douban.com/personage/36583074/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "演员 Actor (饰 Gabriel's Henchman)",
        "character": "(饰 Gabriel's Henchman)",
        "works": [
          "海王",
          "1917",
          "碟中谍7：致命清算"
        ]
      },
      {
        "name": "托马斯·帕雷德斯 Tomás Paredes",
        "link": "https://www.douban.com/personage/36784227/",
        "image": "https://img3.doubanio.com/view/personage/m/public/daa44104e3de62fb5169ca31755b67b2.jpg",
        "role": "演员 Actor (饰 Hagar (as Tomas Paredes))",
        "character": "(饰 Hagar (as Tomas Paredes)",
        "works": [
          "碟中谍8：最终清算",
          "王牌特工：源起",
          "阿盖尔：神秘特工"
        ]
      },
      {
        "name": "雷内·弗拉贝尔 Rene Vrabel",
        "link": "https://www.douban.com/personage/37378167/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "演员 Actor (饰 Gabriel's Henchman)",
        "character": "(饰 Gabriel's Henchman)",
        "works": [
          "碟中谍8：最终清算",
          "边缘世界",
          "航班蛛患"
        ]
      },
      {
        "name": "伊戈尔·卡尔波维奇 Igor Karpovich",
        "link": "https://www.douban.com/personage/37378168/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "演员 Actor (饰 Gabriel's Henchman)",
        "character": "(饰 Gabriel's Henchman)",
        "works": [
          "碟中谍8：最终清算"
        ]
      }
    ],
    "编剧 Writer": [
      {
        "name": "克里斯托弗·麦奎里 Christopher McQuarrie",
        "link": "https://www.douban.com/personage/27504593/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p1535912054.09.jpg",
        "role": "编剧 Writer",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "明日边缘",
          "碟中谍5：神秘国度"
        ]
      },
      {
        "name": "埃里克·延德雷森 Erik Jendresen",
        "link": "https://www.douban.com/personage/27571929/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "编剧 Writer",
        "character": "",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算",
          "兄弟连"
        ]
      },
      {
        "name": "布鲁斯·盖勒 Bruce Geller",
        "link": "https://www.douban.com/personage/27234108/",
        "image": "https://img3.doubanio.com/view/celebrity/m/public/p1432739964.53.jpg",
        "role": "原著作者 Based on",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "碟中谍4",
          "碟中谍5：神秘国度"
        ]
      }
    ],
    "制片人 Producer": [
      {
        "name": "汤姆·克鲁斯 Tom Cruise",
        "link": "https://www.douban.com/personage/27260203/",
        "image": "https://img9.doubanio.com/view/personage/m/public/341b1e42f0d9e68426c414744e7ebd34.jpg",
        "role": "制片人 Producer",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "雨人",
          "明日边缘"
        ]
      },
      {
        "name": "克里斯托弗·麦奎里 Christopher McQuarrie",
        "link": "https://www.douban.com/personage/27504593/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p1535912054.09.jpg",
        "role": "制片人 Producer",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "明日边缘",
          "碟中谍5：神秘国度"
        ]
      },
      {
        "name": "大卫·埃里森 David Ellison",
        "link": "https://www.douban.com/personage/27220121/",
        "image": "https://img3.doubanio.com/view/celebrity/m/public/p1372846967.43.jpg",
        "role": "制片人 Producer",
        "character": "",
        "works": [
          "碟中谍4",
          "碟中谍5：神秘国度",
          "僵尸世界大战"
        ]
      },
      {
        "name": "唐·格兰杰 Don Granger",
        "link": "https://www.douban.com/personage/35503012/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "制片人 Producer",
        "character": "",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算",
          "峡谷"
        ]
      },
      {
        "name": "达娜·哥德伯格 Dana Goldberg",
        "link": "https://www.douban.com/personage/35503060/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "制片人 Producer",
        "character": "",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算",
          "峡谷"
        ]
      },
      {
        "name": "克里斯·布洛克 Chris Brock",
        "link": "https://www.douban.com/personage/36583052/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "制片人 Producer",
        "character": "",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算"
        ]
      }
    ],
    "音乐 Music Department": [
      {
        "name": "塞西尔·图尔内萨克 Cecile Tournesac",
        "link": "https://www.douban.com/personage/37358482/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "音乐总监 Music Supervisor",
        "character": "",
        "works": [
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "麦克斯·阿鲁日 Max Aruj",
        "link": "https://www.douban.com/personage/37358480/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "作曲 Composer",
        "character": "",
        "works": [
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "阿尔菲·戈德弗雷 Alfie Godfrey",
        "link": "https://www.douban.com/personage/37358481/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "作曲 Composer",
        "character": "",
        "works": [
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "拉罗·斯齐弗林 Lalo Schifrin",
        "link": "https://www.douban.com/personage/27224940/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p11868.jpg",
        "role": "《碟中谍》主题曲",
        "character": "",
        "works": [
          "碟中谍8：最终清算",
          "红龙",
          "尖峰时刻2"
        ]
      }
    ],
    "摄影 Camera Department": [
      {
        "name": "弗雷泽·塔加特 Fraser Taggart",
        "link": "https://www.douban.com/personage/35195918/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "摄影 Cinematographer",
        "character": "",
        "works": [
          "碟中谍7：致命清算",
          "碟中谍8：最终清算",
          "机器人帝国"
        ]
      }
    ],
    "剪辑 Editorial Department": [
      {
        "name": "埃迪·汉密尔顿 Eddie Hamilton",
        "link": "https://www.douban.com/personage/30480306/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p1620727049.58.jpg",
        "role": "剪辑 Editor",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "王牌特工：特工学院",
          "王牌特工2：黄金圈"
        ]
      }
    ],
    "选角 Casting Department": [
      {
        "name": "明迪·马丁 Mindy Marin",
        "link": "https://www.douban.com/personage/27519281/",
        "image": "https://img1.doubanio.com/view/celebrity/m/public/p1536057894.8.jpg",
        "role": "选角 Casting",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "战争之王",
          "变脸"
        ]
      }
    ],
    "美术 Art Department": [
      {
        "name": "加里·弗里曼 Gary Freeman",
        "link": "https://www.douban.com/personage/27512435/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "艺术指导 Production Designer",
        "character": "",
        "works": [
          "沉睡魔咒",
          "碟中谍7：致命清算",
          "碟中谍8：最终清算"
        ]
      }
    ],
    "服装 Costume Department": [
      {
        "name": "吉尔·泰勒 Jill Taylor",
        "link": "https://www.douban.com/personage/27527272/",
        "image": "https://img3.doubanio.com/view/personage/m/public/15bea4d9f6b80250ea37db0b2cd83817.jpg",
        "role": "服装设计 Costume Designer",
        "character": "",
        "works": [
          "遇见你之前",
          "碟中谍7：致命清算",
          "赛末点"
        ]
      }
    ],
    "副导演 Assistant Director": [
      {
        "name": "韦德·伊斯特伍德 Wade Eastwood",
        "link": "https://www.douban.com/personage/27504781/",
        "image": "https://img9.doubanio.com/view/celebrity/m/public/p1545188879.56.jpg",
        "role": "B组导演 Second Unit Director",
        "character": "",
        "works": [
          "盗梦空间",
          "星际穿越",
          "碟中谍6：全面瓦解"
        ]
      }
    ],
    "视觉特效 Visual Effects": [
      {
        "name": "亚历克斯·伍特克 Alex Wuttke",
        "link": "https://www.douban.com/personage/27533248/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "视效总监 visual effects supervisor: ILM",
        "character": "",
        "works": [
          "2012",
          "蝙蝠侠：侠影之谜",
          "碟中谍8：最终清算"
        ]
      }
    ],
    "动作特技 Action or Stunts": [
      {
        "name": "韦德·伊斯特伍德 Wade Eastwood",
        "link": "https://www.douban.com/personage/27504781/",
        "image": "https://img9.doubanio.com/view/celebrity/m/public/p1545188879.56.jpg",
        "role": "特技统筹 Stunt Coordinator",
        "character": "",
        "works": [
          "盗梦空间",
          "星际穿越",
          "碟中谍6：全面瓦解"
        ]
      },
      {
        "name": "马修·范·里夫 Matthew Van Leeve",
        "link": "https://www.douban.com/personage/36958460/",
        "image": "https://img1.doubanio.com/view/personage/m/public/79827ef9c0e54a6756fbca91cce1d15d.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "复仇者联盟2：奥创纪元",
          "疯狂的麦克斯4：狂暴之路",
          "生化危机：终章"
        ]
      },
      {
        "name": "鲁道夫·弗尔巴 Rudolf Vrba",
        "link": "https://www.douban.com/personage/37098411/",
        "image": "https://img1.doubanio.com/view/personage/m/public/e55c0dd8a371c36c47dd842b64217a4d.jpg",
        "role": "动作指导 fight coordinator",
        "character": "",
        "works": [
          "美国队长",
          "王牌特工：特工学院",
          "僵尸世界大战"
        ]
      },
      {
        "name": "伊扎伊亚·阿杜亨 Izaiah Aduhene",
        "link": "https://www.douban.com/personage/36986586/",
        "image": "https://img1.doubanio.com/view/personage/m/public/9aa634d71cc87625790f6bd2e0de9099.jpg",
        "role": "替身 Stunt Double",
        "character": "",
        "works": [
          "死侍与金刚狼",
          "碟中谍7：致命清算",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "詹姆斯·昂斯沃斯 James Unsworth",
        "link": "https://www.douban.com/personage/37055507/",
        "image": "https://img1.doubanio.com/view/personage/m/public/639bcad11d11e7a86a3d1e3aa4c29d0b.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "灰姑娘",
          "碟中谍7：致命清算",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "基特·伯登 Kit Burden",
        "link": "https://www.douban.com/personage/36996436/",
        "image": "https://img3.doubanio.com/view/personage/m/public/3166c4eea51fd35ed9e02dfcbf2f95aa.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "死侍与金刚狼",
          "速度与激情10",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "卡利·内勒 Cali Nelle",
        "link": "https://www.douban.com/personage/27554469/",
        "image": "https://img9.doubanio.com/view/celebrity/m/public/p1506305627.04.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "银河护卫队",
          "王牌特工：特工学院",
          "变形金刚5：最后的骑士"
        ]
      },
      {
        "name": "蒂莉·鲍威尔 Tilly Powell",
        "link": "https://www.douban.com/personage/37270857/",
        "image": "https://img3.doubanio.com/view/personage/m/public/101099262678a347cbbca8ba884e01a7.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "哈利·波特与死亡圣器(下)",
          "复仇者联盟2：奥创纪元"
        ]
      },
      {
        "name": "詹姆斯·艾普斯 James Apps",
        "link": "https://www.douban.com/personage/37164677/",
        "image": "https://img3.doubanio.com/view/personage/m/public/dd294665265f0eb6219810298e81a5aa.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "沉睡魔咒2",
          "新蝙蝠侠",
          "碟中谍7：致命清算"
        ]
      },
      {
        "name": "麦克·斯诺 Mike Snow",
        "link": "https://www.douban.com/personage/37176873/",
        "image": "https://img2.doubanio.com/view/personage/m/public/96f9853771be22a75c8c52b76d3027de.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "加勒比海盗5：死无对证",
          "碟中谍8：最终清算",
          "毒液2"
        ]
      },
      {
        "name": "博格丹·库姆沙茨基 Bogdan Kumshatsky",
        "link": "https://www.douban.com/personage/37176872/",
        "image": "https://img9.doubanio.com/view/personage/m/public/efe39c062b55b42b5cb5ab502281ab46.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "1917",
          "速度与激情：特别行动",
          "死侍与金刚狼"
        ]
      },
      {
        "name": "大卫·奎南·布朗 David Guinan-Browne",
        "link": "https://www.douban.com/personage/37176254/",
        "image": "https://img1.doubanio.com/view/personage/m/public/3fcc0be28913e64c28d1df389c4e38a8.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "黑衣人：全球追缉",
          "碟中谍8：最终清算",
          "斯特莱克探案集：墨黑之心"
        ]
      },
      {
        "name": "克雷顿·格洛佛 Clayton Grover",
        "link": "https://www.douban.com/personage/37173830/",
        "image": "https://img2.doubanio.com/view/personage/m/public/eacc5df7d33ffdc19828503e44709541.jpg",
        "role": "特技表演者 stunt performer",
        "character": "",
        "works": [
          "1917",
          "速度与激情：特别行动",
          "速度与激情9"
        ]
      },
      {
        "name": "卢克·戈麦斯 Luke Gomes",
        "link": "https://www.douban.com/personage/37173826/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "西蒙·佩吉替身 stunt double: Simon Pegg",
        "character": "",
        "works": [
          "王牌特工2：黄金圈",
          "碟中谍8：最终清算",
          "人之怒"
        ]
      },
      {
        "name": "杰丝·甘博 Jess Gamble",
        "link": "https://www.douban.com/personage/37164656/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "水下特技替身 underwater stunt double",
        "character": "",
        "works": [
          "王牌特工：特工学院",
          "帕丁顿熊2",
          "碟中谍5：神秘国度"
        ]
      },
      {
        "name": "乔尔·阿德里安 Joel Adrian",
        "link": "https://www.douban.com/personage/35419167/",
        "image": "https://img3.doubanio.com/view/celebrity/m/public/pAtrfmiVCqSUcel_avatar_uploaded1617203019.62.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "功夫瑜伽",
          "速度与激情9",
          "新蝙蝠侠"
        ]
      },
      {
        "name": "艾略特·科林斯 Elliot Collins",
        "link": "https://www.douban.com/personage/37270855/",
        "image": "https://img9.doubanio.com/view/personage/m/public/66d8edd0695aa07dd1edfb5ee97b8c35.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "芭比",
          "速度与激情10",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "查拉采恩·卡贝罗 Chalatsane Kabelo",
        "link": "https://www.douban.com/personage/37351931/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "武师(南非) stunts: South Africa",
        "character": "(南非)",
        "works": [
          "疯狂的麦克斯4：狂暴之路",
          "生化危机：终章",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "托拉·比施 Tola Bishi",
        "link": "https://www.douban.com/personage/37346319/",
        "image": "https://img1.doubanio.com/view/personage/m/public/e9ef8524afb6e85cfd3f12f03727a05c.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "碟中谍8：最终清算",
          "龙之家族",
          "猎神：冬日之战"
        ]
      },
      {
        "name": "路易斯·萨姆斯 Louis Samms",
        "link": "https://www.douban.com/personage/37270971/",
        "image": "https://img1.doubanio.com/view/personage/m/public/11575d397e8389e2f1f1f518e09f0d1d.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "银河护卫队",
          "雷神2：黑暗世界",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "庄·斯雷尔 Jon Slayer",
        "link": "https://www.douban.com/personage/37270969/",
        "image": "https://img9.doubanio.com/view/personage/m/public/4210156d5f4b85441e8cdd4802b23846.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "芭比",
          "新蝙蝠侠",
          "黑寡妇"
        ]
      },
      {
        "name": "阿兰·休伊特 Allan Hewitt",
        "link": "https://www.douban.com/personage/37270933/",
        "image": "https://img3.doubanio.com/view/personage/m/public/fc63220d8dd9cc2c7424e64da55b6c8a.jpg",
        "role": "极限跳伞特技统筹 skydiving coordinator",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "蝙蝠侠：侠影之谜",
          "碟中谍7：致命清算"
        ]
      },
      {
        "name": "卢克·斯科特 Luke Scott",
        "link": "https://www.douban.com/personage/37270948/",
        "image": "https://img3.doubanio.com/view/personage/m/public/69a3cd9ca2540090be5ae5b8fc134adf.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "蜘蛛侠：英雄远征",
          "王牌特工：特工学院",
          "1917"
        ]
      },
      {
        "name": "雅戈·威廉姆斯 Jahgo Williams",
        "link": "https://www.douban.com/personage/37270947/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "碟中谍8：最终清算",
          "惊奇队长2",
          "亚瑟王：斗兽争霸"
        ]
      },
      {
        "name": "汤姆·罗杰斯 Tom Rodgers",
        "link": "https://www.douban.com/personage/37270936/",
        "image": "https://img3.doubanio.com/view/personage/m/public/c0bc28b091a9edf6d785960fb1732922.jpg",
        "role": "替身 Stunt Double",
        "character": "",
        "works": [
          "头号玩家",
          "复仇者联盟2：奥创纪元",
          "雷神2：黑暗世界"
        ]
      },
      {
        "name": "基尔兰·克拉克 Kieran Clarke",
        "link": "https://www.douban.com/personage/37270918/",
        "image": "https://img1.doubanio.com/view/personage/m/public/5c1c5f5a2bca8e0b396f7b67d39ef590.jpg",
        "role": "特技车手 stunt driver",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "碟中谍5：神秘国度",
          "变形金刚5：最后的骑士"
        ]
      },
      {
        "name": "戴夫·凡·泽尔 Dave Van Zeyl",
        "link": "https://www.douban.com/personage/37270946/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "威亚组领班 key stunt rigger",
        "character": "",
        "works": [
          "碟中谍6：全面瓦解",
          "速度与激情7",
          "X战警：逆转未来"
        ]
      },
      {
        "name": "史蒂薇·帕克 Stevie Parker",
        "link": "https://www.douban.com/personage/37359808/",
        "image": "https://img1.doubanio.com/view/personage/m/public/cf5fb7e3b8a85154859a631148b2ff8b.jpg",
        "role": "辅助特技 utility stunts",
        "character": "",
        "works": [
          "速度与激情10",
          "碟中谍8：最终清算",
          "巨齿鲨2：深渊"
        ]
      },
      {
        "name": "斯特凡·米哈拉切 Stefan Mihalache",
        "link": "https://www.douban.com/personage/37359795/",
        "image": "https://img2.doubanio.com/view/personage/m/public/8ce4c346e47ca430935dcd26149ce6c1.jpg",
        "role": "辅助特技 utility stunts",
        "character": "",
        "works": [
          "碟中谍8：最终清算",
          "闪电侠",
          "蚁人与黄蜂女：量子狂潮"
        ]
      },
      {
        "name": "查理·宝莱特 Charlie Pawlett",
        "link": "https://www.douban.com/personage/37366225/",
        "image": "https://img1.doubanio.com/view/personage/m/public/16b0cbc07d58ac17c38fc4e83b096deb.jpg",
        "role": "武师 Stunts",
        "character": "",
        "works": [
          "蝙蝠侠：黑暗骑士",
          "神奇动物：格林德沃之罪",
          "神奇动物在哪里"
        ]
      },
      {
        "name": "彼得·阿尔伯蒂 Peter Alberti",
        "link": "https://www.douban.com/personage/37373981/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "特技表演 stunt performer",
        "character": "",
        "works": [
          "头号玩家",
          "星球大战7：原力觉醒",
          "碟中谍8：最终清算"
        ]
      },
      {
        "name": "凯尔·弗里曼特尔 Kyle Freemantle",
        "link": "https://www.douban.com/personage/36583074/",
        "image": "https://img1.doubanio.com/f/vendors/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/personage-default-medium.png",
        "role": "核心特技团队 core stunt team",
        "character": "",
        "works": [
          "海王",
          "1917",
          "碟中谍7：致命清算"
        ]
      },
      {
        "name": "乔纳森·科恩 Jonathan Cohen",
        "link": "https://www.douban.com/personage/37374018/",
        "image": "https://img3.doubanio.com/view/personage/m/public/f0b1aa4f2e2634dcc2871fcb312909ba.jpg",
        "role": "辅助特技 utility stunt",
        "character": "",
        "works": [
          "头号玩家",
          "哈利·波特与死亡圣器(下)",
          "复仇者联盟2：奥创纪元"
        ]
      }
    ]
  },
  "imdb_rating": "7.2",
  "imdb_votes": "221523",
  "format": "[img]https://img2.pixhost.to/images/6519/705755077_30433456.jpg[/img]\n\n◎片　　名　碟中谍8：最终清算\n◎译　　名　Mission: Impossible - The Final Reckoning / 碟中谍8 / 不可能的任务：最终清算(台) / 职业特工队：最终清算(港) / 碟中谍8：致命清算(下)\n　　　　　　Mission: Impossible – Dead Reckoning Part Two / Mission: Impossible 8 / MI8\n◎年　　代　2025\n◎产　　地　美国\n◎类　　别　动作 / 惊悚 / 冒险\n◎语　　言　英语 / 法语\n◎上映日期　2025-05-30(中国大陆) / 2025-05-14(戛纳国际电影节) / 2025-05-23(美国)\n◎IMDb评分  7.2/10 (221523 人评价)\n◎IMDb链接  https://www.imdb.com/title/tt9603208/\n◎豆瓣评分　7.6/10 (268158 人评价)\n◎豆瓣链接　https://movie.douban.com/subject/30433456/\n◎片　　长　170分钟\n◎导　　演　韦德·伊斯特伍德 Wade Eastwood\n◎编　　剧　克里斯托弗·麦奎里 Christopher McQuarrie / 埃里克·延德雷森 Erik Jendresen / 布鲁斯·盖勒 Bruce Geller\n◎主　　演　汤姆·克鲁斯 Tom Cruise (饰 伊森·亨特 Ethan Hunt)\n　　　　　　海莉·阿特维尔 Hayley Atwell (饰 格蕾丝 Grace)\n　　　　　　文·瑞姆斯 Ving Rhames (饰 卢瑟 Luther Stickell)\n　　　　　　西蒙·佩吉 Simon Pegg (饰 班吉 Benji Dunn)\n　　　　　　埃塞·莫拉雷斯 Esai Morales (饰 盖布瑞尔 Gabriel)\n　　　　　　庞·克莱门捷夫 Pom Klementieff (饰 帕丽斯 Paris)\n　　　　　　亨利·科泽尼 Henry Czerny (饰 尤金·基特里奇 Eugene Kittridge)\n　　　　　　霍特·麦克卡兰尼 Holt McCallany (饰 塞林 Secretary of Defense Bernstein)\n　　　　　　珍妮特·麦克蒂尔 Janet McTeer (饰 沃尔特斯 Walters)\n　　　　　　尼克·奥弗曼 Nick Offerman (饰 西德尼 Sydney)\n　　　　　　汉娜·沃丁厄姆 Hannah Waddingham (饰 尼丽 Admiral Neely)\n　　　　　　特拉梅尔·提尔曼 Tramell Tillman (饰 布莱索 Captain Bledsoe)\n　　　　　　安吉拉·贝塞特 Angela Bassett (饰 艾莉卡·史隆 Erica Sloan)\n　　　　　　谢伊·惠格姆 Shea Whigham (饰 布里格斯 Jasper Briggs)\n　　　　　　格雷格·泰山·戴维斯 Greg Tarzan Davis (饰 德加 Degas)\n　　　　　　查尔斯·帕内尔 Charles Parnell (饰 理查兹 Richards)\n　　　　　　马克·加蒂斯 Mark Gatiss (饰 埃斯特朗 Angstrom)\n　　　　　　罗尔夫·萨克森 Rolf Saxon (饰 威廉·多诺利 William Donloe)\n　　　　　　露西·图卢加鲁克 Lucy Tulugarjuk (饰 塔蓓萨 Tapeesa)\n　　　　　　加利·艾尔维斯 Cary Elwes (饰 Denlinger)\n　　　　　　凯蒂·M·奥布莱恩 Katy M. O'Brian (饰 Kodiak)\n　　　　　　帕沙·D·林奇尼科夫 Pasha D. Lychnikoff (饰 Captain Koltsov)\n　　　　　　艾琳·巴特尔 Erin Battle (饰 Sergeant Armstrong)\n　　　　　　托米·厄尔·詹金斯 Tommie Earl Jenkins (饰 Colonel Burdick)\n　　　　　　迪娜·特鲁迪 Deena Trudy (饰 Agent)\n　　　　　　罗伯·德兰尼 Rob Delaney\n　　　　　　凯尔·弗里曼特尔 Kyle Freemantle (饰 Gabriel's Henchman)\n　　　　　　托马斯·帕雷德斯 Tomás Paredes (饰 Hagar (as Tomas Paredes)\n　　　　　　雷内·弗拉贝尔 Rene Vrabel (饰 Gabriel's Henchman)\n　　　　　　伊戈尔·卡尔波维奇 Igor Karpovich (饰 Gabriel's Henchman)\n\n◎简　　介\n\n　　超级人工智能“智体”即将引爆全球核弹危机，把世界推向毁灭边缘。而伊森·亨特（汤姆·克鲁斯 饰）和他的IMF小队在上次行动中遭遇重创，团队濒临分崩离析。虽然伊森已获得关闭“智体”的钥匙，但要彻底消灭“智体”，完成这一拯救全人类的终极任务，仍需要IMF小队齐心协力突破重重困难。他们不仅要面对全知全能又无影无形的“智体”与其手下盖布瑞尔（埃塞·莫拉雷斯 饰）的百般阻拦，还要解决来自过去的种种恩怨。每个抉择都关乎信念与命运，等待伊森和IMF小队的，究竟是怎样的结局？\n\n◎获奖情况\n\n　　第83届金球奖 (2026)\n　　电影类 电影和票房成就奖(提名)\n\n　　第49届日本电影学院奖 (2026)\n　　最佳外语片(提名)\n\n　　第32届美国演员奖 (2026)\n　　电影奖 电影最佳特技群戏\n\n　　第31届美国评论家选择电影奖 (2026)\n　　最佳视觉效果(提名) 亚历克斯·伍特克 / 克里斯汀·霍尔 / 杰夫·萨瑟兰 / 伊安·洛\n　　最佳特技设计 韦德·伊斯特伍德\n",
  "tier": "svip",
  "usage": {
    "limits": {
      "key_per_min": 300,
      "key_per_day": 100000,
      "ip_per_min": 300,
      "ip_per_day": 100000
    },
    "used": {
      "key_min": 1,
      "key_day": 1,
      "ip_min": 1,
      "ip_day": 1
    }
  }
}
```

（_实际响应还进一步包括海量的 `cast`、`writer`、`full_celebrities` 核心数组集。_）

---

## 📜 声明与开源协议

版权所有 &copy; DoubanInfo Team。
使用即代表您已同意平台的 [服务条款](https://doubaninfo.com/terms.php) 及 [隐私政策](https://doubaninfo.com/privacy.php)。请自觉履行良好的请求礼仪，严禁恶意消耗资源。
