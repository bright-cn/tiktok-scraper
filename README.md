# Python 版 TikTok 爬虫工具

使用 Python 将 TikTok 个人资料、视频、评论和 TikTok Shop 商品抓取为 JSON。无需登录 TikTok，无需浏览器。基于 [Bright Data TikTok 爬虫工具 API](https://www.bright.cn/products/web-scraper/tiktok?utm_source=github&utm_medium=referral&utm_content=tiktok-scraper) 构建。新账户每月可获得 5,000 个免费积分。

[![实时检查](https://github.com/bright-cn/tiktok-scraper/actions/workflows/live.yml/badge.svg)](https://github.com/bright-cn/tiktok-scraper/actions/workflows/live.yml) [![最后验证](https://img.shields.io/badge/last%20verified-9%20Nov%202026-brightgreen)](https://github.com/bright-cn/tiktok-scraper/actions/workflows/live.yml) <!-- verified: rewritten by the daily run -->

[快速开始](#如何使用-python-抓取-tiktok-数据) · [无需登录](#无需登录或访问令牌即可抓取-tiktok) · [CLI](#批量-tiktok-爬虫-cli) · [端点](#抓取-tiktok-帖子个人资料评论和-tiktok-shop) · [数据](#tiktok-帖子数据字段json) · [错误](#tiktok-爬虫错误) · [MCP 和 Claude Code](#tiktok-mcp-服务器和-claude-code-技能) · [免费积分](#这个-tiktok-爬虫免费吗) · [文档](https://docs.brightdata.com/products/scrapers/tiktok/introduction) · [支持](#支持)

使用 Python 将 TikTok 个人资料、帖子、评论和 TikTok Shop 商品获取为 JSON。无需登录 TikTok，无需浏览器。基于 [Bright Data TikTok 爬虫工具 API](https://www.bright.cn/products/web-scraper/tiktok?utm_source=github&utm_medium=referral&utm_content=tiktok-scraper) 构建。

使用 [Bright Data Python SDK](https://github.com/bright-cn/sdk-python)。完整 API 文档：[TikTok 爬虫工具 API](https://docs.brightdata.com/products/scrapers/tiktok/introduction)。

此外还提供：用于获取近期帖子的一键式 CLI，以及完全不需要 Python 的 [Bright Data CLI](#tiktok-mcp-服务器和-claude-code-技能)。

## 如何使用 Python 抓取 TikTok 数据

需要 Python 3.10 或更高版本。

```bash
pip install brightdata-sdk
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。项目根目录中的 `.env` 文件也可以使用，前提是虚拟环境位于项目内。

也可以跳过手动设置令牌。运行一次 `npx -p @brightdata/cli bdata login`：它会打开浏览器，此后 SDK 会自行找到已存储的凭据，供你以及在该终端中工作的任何编码智能体使用。智能体无法点击完成登录，因此请先自行操作。同一个 CLI 也可以直接抓取 TikTok；请参阅 [TikTok MCP 服务器和 Claude Code 技能](#tiktok-mcp-服务器和-claude-code-技能)。

还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户每月可获得 [5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.tiktok.posts_by_profile(
        "https://www.tiktok.com/@tiktok", num_of_posts=5, timeout=900
    )
    if not result.success:
        raise SystemExit(f"{result.status}: {result.error}")
    for post in result.data:
        print(post["digg_count"], post["comment_count"], post["url"])
```

```
12300 3222 https://www.tiktok.com/@tiktok/video/7692114317151423775
12000 5091 https://www.tiktok.com/@tiktok/video/7694797676306615583
9428 1391 https://www.tiktok.com/@tiktok/video/7691013160425327903
14800 3613 https://www.tiktok.com/@tiktok/video/7693184538704416031
11200 1821 https://www.tiktok.com/@tiktok/video/7691450773619625247
```

预计需要五到十分钟：API 会运行发现作业，SDK 会等待其完成。每篇帖子消耗一个[积分](https://www.bright.cn/pricing/web-scraper)。`digg_count` 是 TikTok 对点赞数使用的名称。

该代码片段中的三项设置不是可选的。

`auto_create_zones=False` 会阻止 SDK 在启动时创建区域。这些区域供网络解锁器和 SERP 使用，它们是此爬虫从不涉及的另外两款 Bright Data 产品；没有付款方式的账户创建区域时会失败（[sdk-python#57](https://github.com/brightdata/sdk-python/issues/57)）。

`timeout=900`。SDK 默认在 240 秒后放弃。2026 年 10 月 20 日，此调用需要 303 到 532 秒，有一次甚至超过 600 秒。到 240 秒时，它会返回 `success=False`，且不返回数据。

`num_of_posts` 会限制运行数量和费用。如果没有它，发现操作会采集该账户的所有帖子。

## 无需登录或访问令牌即可抓取 TikTok

TikTok 爬虫工具 API 不需要 TikTok 账户、TikTok 登录信息或 TikTok Research API 访问权限。唯一的凭据是 Bright Data API 密钥，可将其设置为 `BRIGHTDATA_API_TOKEN`，或通过 `bdata login` 存储。

## 批量 TikTok 爬虫 CLI

此存储库中的命令会同时为多个账户运行 Python 快速开始示例，并写入一个 JSON 文件。

```bash
pip install git+https://github.com/bright-cn/tiktok-scraper
tiktok-scraper tiktok khaby.lame
```

```
Fetching up to 5 recent posts per account, for: tiktok, khaby.lame
Usually two to six minutes each. One credit per post, 5,000 free per month.
asking  @tiktok...
got     @tiktok: 5 posts
asking  @khaby.lame...
got     @khaby.lame: 5 posts

Saved 10 posts as JSON to tiktok.json (44 fields per post)
```

在终端中，`asking` 行会被原地更新的加载动画和计时器取代，因此你可以看到它正在运行以及已经运行了多长时间。

```
--limit N    posts per account, default 5, minimum 1
--out PATH   output file, default tiktok.json
```

`python -m tiktok_scraper` 也可以使用。

也可以导入它而不是直接运行，以便按账户获得 `ok`、`note` 和 `error`，而不是原始行。`scrape` 不会因为某一个无效账户而引发异常；读取 `posts` 前，请先检查 `ok`：

```python
from tiktok_scraper import scrape

for outcome in scrape(["tiktok", "zz_not_a_real_account_zz"], limit=1):
    if outcome.ok:
        print(f"{outcome.handle}: {len(outcome.posts)} posts")
    else:
        print(f"{outcome.handle} failed: {outcome.error}")
```

```
tiktok: 1 posts
zz_not_a_real_account_zz failed: User is banned
```

不存在的账户会返回 "User is banned"。在搜索时间段内没有内容的账户会成功返回，但没有帖子。原因位于 `note`，而不是 `error`。

## 抓取 TikTok 帖子、个人资料、评论和 TikTok Shop

Bright Data TikTok 爬虫工具 API 有 12 个端点，每个端点都在 [API 参考](https://docs.brightdata.com/api-reference/scrapers/social-media-apis/tiktok-profiles-collect-by-url)中拥有单独页面。下面的每个代码片段都是完整的，只需要 `brightdata-sdk`；原始 HTTP 则什么都不需要：按原样粘贴即可。每个片段都会在每周一的 Actions 中运行，每隔一天还会执行一次规模较小的检查，顶部徽章显示的是最新结果。

| 你拥有 | 你想要 | 方法 |
| --- | --- | --- |
| 个人资料 URL | 近期帖子，44 个字段 | `client.search.tiktok.posts_by_profile(url, num_of_posts=5, timeout=900)` |
| 个人资料 URL | 近期帖子，36 个字段，速度更快 | 原始 HTTP，按个人资料获取帖子的 Fast API。SDK 的 `posts_by_profile_fast` 不发送 `num_of_posts`，因此会对每篇帖子计费 |
| 个人资料 URL | 个人资料 | `client.scrape.tiktok.profiles(url)` |
| TikTok 搜索 URL | 匹配的个人资料 | 原始 HTTP，个人资料 `discover_by=search_url` |
| 帖子 URL | 对应帖子 | `client.scrape.tiktok.posts([url, url])` |
| 关键词 | 匹配的帖子 | `client.search.tiktok.posts_by_keyword("street food", num_of_posts=2, timeout=900)` |
| 标签、音乐或探索页 URL | 该页面上的帖子 | 原始 HTTP，帖子 `discover_by=url` |
| 帖子 URL | 其评论 | `client.scrape.tiktok.comments(url)`，每条评论消耗一个积分 |
| TikTok Shop 商品 URL | 该商品 | 原始 HTTP，Shop 采集 |
| Shop 类别 URL、关键词或店铺 URL | 商品 | 原始 HTTP，Shop `discover_by=category`、`keyword` 或 `shop` |

所有这些都是异步作业。SDK 会触发作业、进行轮询，并在准备就绪后返回。之所以会出现错误行（例如上面的 "User is banned"），是因为 SDK 使用 `include_errors=true` 请求返回它们。API 默认关闭此选项。

`search` 调用会在 `result.data` 中返回列表。使用一个 URL 的 `scrape` 调用会以字典形式返回一条记录，使用多个 URL 时则返回列表。

### 在一个作业中抓取多个 TikTok 账户

URL 列表是一个作业，而不是每个账户一个作业。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.tiktok.posts_by_profile(
        ["https://www.tiktok.com/@tiktok", "https://www.tiktok.com/@khaby.lame"],
        num_of_posts=1,
        timeout=900,
    )
    for post in result.data:
        print(post["account_id"], post["url"])
```

```
khaby.lame https://www.tiktok.com/@khaby.lame/video/7697307919943060767
tiktok https://www.tiktok.com/@tiktok/video/7694797676306615583
```

`account_id` 是用户名。`profile_username` 是显示名称，例如 "TikTok" 或 "Khabane lame"，而不是用户名。

### 立即触发，稍后获取

如果账户数量不止几个，不要让进程阻塞一小时。触发作业、保留快照 ID，并在就绪后获取。快照可下载 30 天。

```python
import time

from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    job = client.scrape.tiktok.posts_trigger("https://www.tiktok.com/@tiktok/video/7680721699171601694")
    print("snapshot:", job.snapshot_id)
    while (status := client.scrape.tiktok.posts_status(job.snapshot_id)) not in ("ready", "failed"):
        time.sleep(5)
    print("status:", status)
    record = client.scrape.tiktok.posts_fetch(job.snapshot_id)[0]
    print("fetched:", record["url"], "likes:", record["digg_count"])
```

```
snapshot: sd_mvgcsmg815mdlpv6kj
status: ready
fetched: https://www.tiktok.com/@tiktok/video/7680721699171601694 likes: 22900
```

### 按日期筛选 TikTok 帖子

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.tiktok.posts_by_profile(
        "https://www.tiktok.com/@tiktok",
        num_of_posts=3,
        start_date="09-01-2026",  # MM-DD-YYYY
        end_date="09-30-2026",
        timeout=900,
    )
    for post in result.data:
        print(post["create_time"], post["play_count"], post["url"])
```

```
2026-09-25T20:13:19.000Z 449600 https://www.tiktok.com/@tiktok/video/7689568478570843422
2026-09-29T17:39:33.000Z 264700 https://www.tiktok.com/@tiktok/video/7691013160425327903
2026-09-30T21:57:30.000Z 346500 https://www.tiktok.com/@tiktok/video/7691450773619625247
```

没有任何内容的时间范围会返回一行，其 `error` 为 "There are no public posts in the profile for the specified period"。该命令会将此行视为零篇帖子，而不是失败。

### 通过 URL 抓取 TikTok 个人资料

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    profile = client.scrape.tiktok.profiles("https://www.tiktok.com/@tiktok").data
    print(profile["account_id"], "followers:", profile["followers"], "videos:", profile["videos_count"])
```

```
tiktok followers: 96400000 videos: 1510
```

### 按关键词抓取 TikTok 帖子

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.tiktok.posts_by_keyword("street food", num_of_posts=2, timeout=900)
    for post in result.data:
        print(post["account_id"], post["play_count"], post["url"])
```

```
hughabroad 6600000 https://www.tiktok.com/@hughabroad/video/7571414421311065346
biteswithlily 11500000 https://www.tiktok.com/@biteswithlily/video/7598108209903127829
```

### 抓取 TikTok 视频的评论

每条评论消耗一个积分，而 `comments()` 无法请求更少的评论。它默认也会在 240 秒后放弃，而一篇有两条评论的帖子曾耗时 243 秒，因此请传入 `timeout=900`。请先检查帖子上的 `comment_count`：本 README 其他位置使用的 TikTok 帖子有超过 5,000 条评论。下面的帖子有两条。要限制大型帖子的评论数量，请使用带 `limit_per_input` 的原始 HTTP；对一篇有 344 条评论的帖子设置 3 后，返回了 3 行。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    url = "https://www.tiktok.com/@cooking__alone/video/7565981443512339742"
    comments = client.scrape.tiktok.comments(url, timeout=900).data
    print(len(comments), "comments; first:", repr(comments[0]["comment_text"][:60]))
```

```
2 comments; first: 'GfcgUZxolnnnmmmmmmmm mmm.'
```

### 通过原始 HTTP 使用 TikTok Shop 爬虫和发现功能

SDK 没有 TikTok Shop 方法，它的其他发现调用也不接受数量限制。这些调用仅使用标准库直接访问 API。此路径只读取 `BRIGHTDATA_API_TOKEN`，不会读取 CLI 登录信息。

```python
import json
import os
import time
import urllib.request

API = "https://api.brightdata.com/datasets/v3"
HEADERS = {
    "Authorization": f"Bearer {os.environ['BRIGHTDATA_API_TOKEN']}",
    "Content-Type": "application/json",
}


def call(path, body=None):
    data = None if body is None else json.dumps(body).encode()
    request = urllib.request.Request(API + path, data=data, headers=HEADERS)
    with urllib.request.urlopen(request) as response:
        return json.load(response)


product = "https://shop.tiktok.com/us/pdp/wireless-bluetooth-earphones-low-latency-gaming-gift/1732339489808880487"
job = call("/trigger?dataset_id=gd_m45m1u911dsa4274pi&include_errors=true", [{"url": product}])
while call(f"/progress/{job['snapshot_id']}")["status"] not in ("ready", "failed"):
    time.sleep(10)
record = call(f"/snapshot/{job['snapshot_id']}?format=json")[0]
if "error" in record:
    raise SystemExit(record["error"])
print(record["title"], record["final_price"], record["currency"], record["store_name"])
```

```
Wireless Bluetooth Earphones, Black TWS Gaming Headset 13.99 USD Saeme
```

其他每个端点都使用相同的 `/trigger` 调用，只需更换数据集、`type=discover_new&discover_by=...` 对和输入。添加 `&limit_per_input=N` 可限制发现数量：如果没有它，关键词或类别发现会返回所有匹配项，并对所有结果计费。以下每个输入均已于 2026-10-20 获实时 API 接受。其中多个输入与 API 参考页面不同，应以实时 API 为准：

| 端点 | dataset_id | discover_by | 输入 |
| --- | --- | --- | --- |
| Shop 商品 | `gd_m45m1u911dsa4274pi` | 无 | `{"url"}`，一个 `shop.tiktok.com/us/pdp/...` URL |
| 按关键词发现 Shop 商品 | `gd_m45m1u911dsa4274pi` | `keyword` | `{"keyword"}`。`num_of_products` 会被拒绝 |
| 按类别发现 Shop 商品 | `gd_m45m1u911dsa4274pi` | `category`，而不是 `category_url` | `{"category_url"}` |
| 按店铺发现 Shop 商品 | `gd_m45m1u911dsa4274pi` | `shop`，而不是 `shop_url` | `{"url"}`，而不是 `shop_url` |
| 按个人资料获取帖子，Fast API，36 个字段 | `gd_m7n5v2gq296pex2f5m` | 无 | `{"url", "num_of_posts"}` |
| 标签或音乐页面上的帖子 | `gd_lu702nij2f790tmv9h` | `url` | `{"URL"}`，且为大写。`url` 会被拒绝 |
| 通过搜索获取个人资料 | `gd_l1villgoiiidt09ci` | `search_url` | `{"search_url", "country"}`，两者均为必填项 |

`bdata pipelines info <dataset_id> <method>` 会输出 API 实际验证的输入字段，以上表格就是通过这种方式检查的。发现操作速度较慢：按关键词和类别发现 Shop 商品时，获取两个商品需要 18 到 20 分钟。

## TikTok 帖子数据字段（JSON）

大多数人需要的字段：

```
url  create_time  description  hashtags  digg_count  comment_count  play_count  account_id
```

`digg_count` 表示点赞数，`collect_count` 表示收藏数，而 `share_count` 以字符串形式返回。代码不会硬编码任何字段列表。API 返回的所有内容都会进入 `result.data`，并写入命令生成的文件。

<!-- fields:start -->
<details>
<summary>全部 44 个字段及其类型和说明</summary>

每天通过 `client.datasets.tiktok_posts.get_metadata()` 根据数据集架构重新生成，因此不会过时。帖子会包含适用于它的字段；示例文件包含这 44 个字段中的全部 44 个。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `url` | url | 唯一帖子链接 |
| `post_id` | text | 每篇帖子的标识信息 |
| `description` | text | 帖子附带的文本 |
| `create_time` | date | 帖子创建时间戳 |
| `digg_count` | number | 收到的点赞数 |
| `share_count` | text | 分享次数 |
| `collect_count` | number | 收藏次数 |
| `comment_count` | number | 评论数 |
| `play_count` | number | 视频播放次数 |
| `video_duration` | number | 视频时长 |
| `hashtags` | array | 包含的标签 |
| `original_sound` | text | 使用的声音（如有） |
| `profile_id` | text | 唯一用户标识符 |
| `profile_username` | text | 用户的用户名 |
| `profile_url` | url | 用户的个人资料链接 |
| `profile_avatar` | url | 用户的头像 |
| `profile_biography` | text | 用户的个人简介 |
| `preview_image` | url | 帖子预览图 |
| `post_type` | text | 帖子类型（视频、图片、文本） |
| `discovery_input` | object | 发现输入值 |
| `offical_item` | boolean | 表示视频是否为官方内容 |
| `secu_id` | text | 个人资料的安全标识符 |
| `original_item` | boolean | 表示视频是否为原创 |
| `shortcode` | text | 视频的短代码 |
| `width` | number | 视频宽度 |
| `ratio` | text | 视频宽高比 |
| `video_url` | url | 视频 URL，生成的 URL 仅在当前会话中有效 |
| `music` | object | 视频所用音乐的信息 |
| `cdn_url` | url | 媒体 URL 链接 |
| `is_verified` | boolean | 表示 TikTok 账户是否已验证（布尔值：True/False） |
| `account_id` | text | 账户 ID |
| `carousel_images` | array | “轮播”类型 TikTok 帖子的图片列表 |
| `tagged_user` | array | 被标记的用户 |
| `profile_followers` | number | 帖子作者的关注者数量 |
| `tt_chain_token` | text | TikTok 用于跟踪和验证用户请求的会话相关令牌 |
| `region` | text | 地区列表示视频的来源国家/地区（例如 US、UK） |
| `commerce_info` | text | 表示帖子是否包含购物、推广或联盟营销内容的信息 |
| `subtitle_url` | url | 字幕 URL |
| `subtitle_format` | text | 字幕格式 |
| `country` | text | 某些个人资料受地理位置限制。请选择一个国家/地区以继续（来自发现操作） |
| `subtitle_info` | array | 字幕信息 |
| `cdn_link` | text | 媒体 URL 链接 |
| `num_share_count` | number | 分享次数 |
| `comments` | array | 帖子的评论 |

</details>
<!-- fields:end -->

<details>
<summary>来自 <code>tiktok-scraper tiktok --limit 1</code> 的真实输出文件开头</summary>

```json
{
  "generated_at": "2026-10-20T07:39:08.192204+00:00",
  "handles": [
    {
      "handle": "tiktok",
      "posts": [
        {
          "url": "https://www.tiktok.com/@tiktok/video/7694797676306615583",
          "post_id": "7694797676306615583",
          "description": "this summer gave us pop songs, glam routines, and a few main-character habits. @McCall Mirabella breaks down this summer's souvenirs \ud83d\udc2c\u2600\ufe0f\nMcCall Mirabella breaks down the key souvenirs of this summer, from the pop songs we can't stop listening to and glam routines to the main-character habits we all adopted. \ud83d\udc2c\u2600\ufe0f\nThis summer was more than just a season; it was a collection of memories and trends that defined the moment. From the songs that were on repeat to the specific routines we all adopted, the summer of 2024 left its mark. McCall Mirabella breaks down the key souvenirs from this summer, capturing the essence of the season.\n\nThe soundtrack of the summer was undeniably pop, with catchy tunes that became the backdrop for countless moments. Alongside the music, we saw a rise in \"glam routines,\" where self-care and beauty became a central part of the daily experience. These weren't just simple tasks; they were rituals that felt special and indulgent.\n\nPerhaps the most defining trend was the adoption of \"main-character habits.\" This concept encourages living life as if you're the protagonist of your own story, embracing confidence, style, and a sense of purpose in everyday actions. McCall Mirabella's breakdown explores how these pop songs, glam routines, and main-character habits came together to create the unique souvenirs of this summer. \ud83d\udc2c\u2600\ufe0f",
          "create_time": "2026-10-09T22:25:11.000Z",
          "digg_count": 12000,
          "share_count": "1260",
          "collect_count": 1229,
          "comment_count": 5094,
          "play_count": 1400000,
          "video_duration": 49,
          "original_sound": "TikTok: original sound",
          "profile_id": "107955",
          "profile_username": "TikTok",
  ...
```

包含一篇帖子及其所有字段的完整文件位于 [examples/sample_output.json](examples/sample_output.json)。

</details>

## TikTok 爬虫错误

| 你看到的内容 | 含义 |
| --- | --- |
| `API token required but not found.` | 在发送任何请求前以状态码 2 退出。请设置令牌。 |
| `failed  @name: User is banned` | 以状态码 1 退出。没有此账户，通常是输入错误。对于不存在的账户，TikTok 会显示 "banned"。 |
| `got     @name: 0 posts, the account has no public posts in the period searched` | 以状态码 0 退出，结果正确。API 会将空时间范围报告为错误行。 |
| `failed  @name: timeout` | 以状态码 1 退出。请求在 900 秒后放弃。请重新运行。 |

任何失败都会以状态码 1 退出，因此可以安全地根据运行结果控制脚本。

在 SDK 中，相同情况如下所示：

| 你看到的内容 | 含义 |
| --- | --- |
| `AuthenticationError: Unauthorized (401)` | 令牌已设置，但不正确。 |
| `result.success` 为 `False`，`result.status` 为 `"timeout"` | SDK 放弃等待，默认超时为 240 秒。传入 `timeout=900`，或重新运行。 |
| `result.data` 中包含 `error` 键的一行 | API 针对某个输入的响应：时间范围为空，或账户不存在。其他行不受影响。 |
| 原始 HTTP 返回 HTTP 400 `validation_error` | API 不接受某个输入字段。消息会指出该字段；请与上表对照。 |

## TikTok MCP 服务器和 Claude Code 技能

无需 Python，也无需安装任何内容。粘贴这两行；第一行会打开一次浏览器，或者在 SSH 和 CI 中使用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines run gd_m7n5v2gq296pex2f5m collect_by_url --url "https://www.tiktok.com/@tiktok" --num_of_posts 2 --pretty
```

CLI 可通过数据集 ID 运行爬虫，支持所有 TikTok 数据集，包括 Shop 和发现功能。`bdata pipelines search tiktok` 会列出它们的 ID 和方法，`--limit-per-input N` 可限制发现数量，默认值为 10。

`npx skills add brightdata/skills` 会让 Claude Code、Cursor 和 Codex 掌握这些命令和文档，之后即可使用自然语言操作。完整指南：[面向编码智能体的 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

使用托管助手且没有终端？[Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)可通过 `search_scrapers` 和 `run_scraper` 等六种工具运行所有 Bright Data 爬虫，包括 TikTok 爬虫。它们默认启用；如只需加载这些工具，请添加 `scrapers` 组：

    https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=scrapers

智能体也可以自行开设账户，无需填写注册表单：[智能体注册](https://brightdata.com/auth.md)。Bright Data 支持的其他所有连接方式，从 LangChain 到 Zapier 和 n8n，均可参阅[集成](https://docs.brightdata.com/integrations/introduction)。

<details>
<summary>CLI 的 TikTok 命令，均已于 2026 年 10 月 20 日运行</summary>

在每条命令前添加 `npx -p @brightdata/cli`，或使用 `npm i -g @brightdata/cli` 一次性安装。每条命令都会输出 JSON，每条记录消耗一个积分。

五个爬虫均使用 `bdata pipelines run <dataset_id> <method>` 运行：个人资料 `gd_l1villgoiiidt09ci`、帖子 `gd_lu702nij2f790tmv9h`、评论 `gd_lkf2st302ap89utw5k`、按个人资料获取帖子的 Fast API `gd_m7n5v2gq296pex2f5m`，以及 TikTok Shop `gd_m45m1u911dsa4274pi`。

```bash
bdata pipelines run gd_l1villgoiiidt09ci collect_by_url "https://www.tiktok.com/@tiktok" --pretty
```

1 条记录，35 个字段，33 秒。`"account_id": "tiktok"`、`"followers": 96300000`。

```bash
bdata pipelines run gd_m7n5v2gq296pex2f5m collect_by_url --url "https://www.tiktok.com/@tiktok" --num_of_posts 2 --pretty
```

2 条记录，34 个字段，70 秒。如果没有 `--num_of_posts`，它会返回该账户的所有帖子，并对所有帖子计费。

```bash
bdata pipelines run gd_lu702nij2f790tmv9h collect_by_url "https://www.tiktok.com/@tiktok/video/7680721699171601694" --format csv -o posts.csv
```

11 分钟。CSV 表头使用与上表相同的字段名称。

```bash
bdata pipelines run gd_lu702nij2f790tmv9h discover_by_keyword --search_keyword "street food" --limit-per-input 2 --pretty
```

2 条记录，41 个字段，6.5 分钟。

```bash
bdata pipelines run gd_lkf2st302ap89utw5k collect_by_url "https://www.tiktok.com/@cooking__alone/video/7565981443512339742" --timeout 900 --pretty
```

2 条记录，16 个字段，2 分钟。每条评论消耗一个积分，因此请先检查帖子的 `comment_count`。未使用 `--timeout 900` 的同一命令曾在 CLI 默认的 600 秒超时后放弃。

```bash
bdata pipelines run gd_m45m1u911dsa4274pi collect_by_url "https://shop.tiktok.com/us/pdp/wireless-bluetooth-earphones-low-latency-gaming-gift/1732339489808880487" --pretty
```

1 个商品，44 个字段，2 分钟。Shop 发现的工作方式相同，可使用 `discover_by_keyword --keyword`、`discover_by_category --category_url` 或 `discover_by_shop`，但获取两个商品需要 18 到 20 分钟，超过 600 秒的默认超时：请添加 `--timeout 1500`，或使用 `--async` 并稍后通过 `bdata pipelines results <snapshot_id>` 获取。

`bdata pipelines info <dataset_id> <method>` 会显示爬虫的输入字段。`bdata pipelines run --help` 会显示 `--format json|csv|ndjson|jsonl`、`-o FILE`、`--timeout` 和 `--async`。

</details>

## 这个 TikTok 爬虫免费吗？

新 Bright Data 账户每月可获得 [5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。帖子、个人资料和 TikTok Shop 商品每条记录消耗一个积分，评论则每条消耗一个积分，因此免费积分每月可覆盖 5,000 篇帖子。超出后的费率请参阅[定价](https://www.bright.cn/pricing/web-scraper)。

## 支持

此存储库中的错误：[提交 issue](https://github.com/bright-cn/tiktok-scraper/issues)，[CONTRIBUTING.md](CONTRIBUTING.md) 说明了需要提供的内容。有关 API、你的账户或积分的任何问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

[Apache 2.0](LICENSE)。

## Node.js 版 TikTok 爬虫工具

同一款 TikTok 爬虫工具也提供 Node.js 版本：[bright-cn/tiktok-scraper-node](https://github.com/bright-cn/tiktok-scraper-node)。

## 更多 Bright Data 开源爬虫工具

以相同方式构建并包含实时验证示例的其他 Python 爬虫：

- [Python 版 Instagram 爬虫工具](https://github.com/bright-cn/instagram-scraper-python)
- [Python 版 YouTube 爬虫工具](https://github.com/bright-cn/youtube-scraper)
- [Python 版 Facebook 爬虫工具](https://github.com/bright-cn/facebook-scraper)
