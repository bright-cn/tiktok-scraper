# 供在此存储库中工作的编码智能体使用

更改任何内容前请先阅读本文。以下每一行都已于 2026-10-20 根据实时 API、已安装的 SDK（`brightdata-sdk` 2.5.2）或 Bright Data CLI 0.4.0 验证。如果 API 参考页面与实时 API 不一致，以此处记录的实时 API 行为为准。

## 身份验证

- SDK 会从环境变量、从其安装文件夹开始向上搜索到的 `.env` 文件（当虚拟环境位于项目内时，即项目根目录），或从 Bright Data CLI 登录信息中读取 `BRIGHTDATA_API_TOKEN`。设置该变量或运行一次 `bdata login`，是唯一需要人工完成的步骤；切勿将密钥粘贴到代码中。
- 始终使用 `SyncBrightDataClient(auto_create_zones=False)` 构造客户端。如果保持启用，SDK 会在启动时尝试创建网络解锁器和 SERP 区域。此存储库从不使用区域，而且没有付款方式的账户会在创建区域时失败。

## API 的行为方式

- 每次 SDK 调用都是异步作业：触发、轮询、返回。发现某个个人资料的近期帖子需要 303 到 532 秒，有一次甚至超过了 600 秒。SDK 默认在 240 秒后放弃，并返回 `success=False`、`status="timeout"`，且不返回任何行。每次发现调用都要传入 `timeout=900`。
- `search.tiktok.posts_by_profile(url, num_of_posts=N)` 返回包含 44 个字段的完整帖子记录。`scrape.tiktok.posts_by_profile_fast(url)` 会更快地返回相同帖子，但只有 36 个字段；然而 SDK 不发送 `num_of_posts`，因此会对该账户的每个帖子计费。不要将其用于有数量上限的运行。
- 一次调用中包含多个 URL 仍是一个作业。应优先采用这种方式，而不是循环调用。对于大型运行，请使用 `posts_trigger`、`posts_status`、`posts_fetch`，并保留快照 ID。
- 每条记录消耗一个积分。每条评论消耗一个积分，而且 SDK 无法限制评论数量：请先检查帖子上的 `comment_count`。使用 `limit_per_input` 的原始 HTTP 可以限制数量；对一篇有 344 条评论的帖子设置 3 后，返回了 3 行。
- SDK 没有 TikTok Shop 方法。Shop 需要使用原始 HTTP 或 CLI。它的其他发现调用 `search.tiktok.profiles` 和 `posts_by_url` 不接受数量限制。
- 不存在的用户名会返回一行，其错误为 "User is banned"。空日期范围会返回一行，其错误为 `EMPTY_WINDOW`，error_code 为 `dead_page`。请匹配消息，而不是错误代码。
- `account_id` 是用户名。`profile_username` 是显示名称。
- 以下是 API 实际验证的输入字段名称。多个 API 参考页面有误：
  - 按 URL 发现帖子使用 `URL`，且为大写。`url` 会返回 400："This input should not contain a url field"。
  - Shop 发现使用 `discover_by=category`、`keyword` 或 `shop`。页面中所写的 `category_url` 和 `shop_url` 会返回 400："Incorrect discovery collector id Available types: category, keyword, shop"。
  - 按关键词发现 Shop 商品使用 `keyword`，并可选用 `domain`。`num_of_products` 会返回 400。
  - 按店铺发现 Shop 商品使用 `url`，而不是 `shop_url`。按类别发现 Shop 商品仅使用 `category_url`。
  - Shop 发现速度较慢：按关键词或类别发现两个商品需要 18 到 20 分钟。
  - 通过搜索 URL 发现个人资料需要 `country`。
  - `bdata pipelines info <dataset_id> <method>` 会输出实时输入架构。
- 架构可能随时更改，恕不另行通知。切勿硬编码字段列表。
- 完整文档索引，每个条目对应一个 `.md` 页面：https://docs.brightdata.com/llms.txt

## 在此处工作

- `pytest` 可离线运行，不需要令牌。`ruff check .` 必须通过。
- CI 会在一台空白机器上使用 README 自身的命令进行安装。每周工作流会针对真实 API 执行 README 中的每个围栏代码块，每日工作流则会运行规模较小的实时检查。代码片段必须完整且可单独粘贴运行，所显示的输出也必须真实。
- README 中的字段表位于 `<!-- fields:start -->` 和 `<!-- fields:end -->` 之间，每日重新生成。不要手动编辑。
- README 顶部的“最后验证”徽章行由每日运行重写。不要手动编辑。
- 保持精简。不要添加重试、去重、调度、数据库、异步示例或并发。
