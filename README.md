# 快手数据接口（Kuaishou API）：视频搜索、评论、用户主页、热榜

**快手数据 API · 快手视频搜索 · 快手评论采集 · 快手用户主页和作品 · 快手热榜。**

这个仓库是 [okdata](https://okdata.online/zh/?src=gh-kuaishou-api) 的快手接口示例：8 个接口在线可用，一个 API Key 调用，按次计费（$0.002 起），不用订阅。不用登录平台账号，不用 Cookie，不开浏览器，所以也不存在账号被风控的问题。

[免费注册拿 Key（送 $0.1 试用额度）](https://okdata.online/zh/signup/?src=gh-kuaishou-api) · [全部接口文档](https://okdata.online/zh/apis/?src=gh-kuaishou-api) · [接入 AI（MCP）](https://okdata.online/zh/mcp/?src=gh-kuaishou-api)

## 接口清单

| 接口 | 说明 | 价格 | 路径 |
|---|---|---|---|
| [快手视频搜索](https://okdata.online/zh/apis/kuaishou/videos/search/) | 按关键词搜视频，可按最新或最多点赞排，可按发布时间和时长筛选：文案、作者、播放、点赞、评论和分享。 | $0.02 / 次 | `GET /v1/kuaishou/videos/search` |
| [快手作品详情](https://okdata.online/zh/apis/kuaishou/videos/get/) | 按 ID 或链接（v.kuaishou.com 短链也行）取一个作品：文案、作者、播放、点赞、评论和分享。 | $0.002 / 次 | `GET /v1/kuaishou/videos/get` |
| [快手作品评论](https://okdata.online/zh/apis/kuaishou/videos/comments/list/) | 作品的一级评论，从新到旧：内容、评论者、点赞、发布时间和 IP 属地。 | $0.002 / 次 | `GET /v1/kuaishou/videos/comments/list` |
| [快手评论回复](https://okdata.online/zh/apis/kuaishou/comments/replies/list/) | 按评论 ID 和所在作品 ID 取一级评论下的回复：内容、回复者、点赞、发布时间和 IP 属地。 | $0.002 / 次 | `GET /v1/kuaishou/comments/replies/list` |
| [快手账号信息](https://okdata.online/zh/apis/kuaishou/users/get/) | 按用户 ID 取账号：名称、快手号、简介、粉丝、关注、作品数、获赞总数、认证和 IP 属地。 | $0.02 / 次 | `GET /v1/kuaishou/users/get` |
| [快手账号搜索](https://okdata.online/zh/apis/kuaishou/users/search/) | 按关键词找账号，可只要认证账号或正在直播的，可按性别筛选或按粉丝数排：用户 ID、名称、快手号、简介、粉丝数和认证。 | $0.02 / 次 | `GET /v1/kuaishou/users/search` |
| [快手账号作品](https://okdata.online/zh/apis/kuaishou/users/videos/list/) | 账号发布的作品，按最新或最热排：文案、播放、点赞、评论和发布时间。 | $0.02 / 次 | `GET /v1/kuaishou/users/videos/list` |
| [快手热榜](https://okdata.online/zh/apis/kuaishou/trends/list/) | 快手热榜：50 个热点和置顶的一条，带热度。 | $0.002 / 次 | `GET /v1/kuaishou/trends/list` |

## 快速开始

1. 在 [okdata 控制台](https://okdata.online/zh/signup/?src=gh-kuaishou-api) 用邮箱注册，拿到 `okd_` 开头的 Key。
2. 调一次接口：

```bash
export OKDATA_API_KEY=okd_...
curl -G "https://api.okdata.online/v1/kuaishou/videos/search" \
  -H "Authorization: Bearer $OKDATA_API_KEY" \
  --data-urlencode 'query=露营装备'
```

Python 版在 [examples/quickstart.py](examples/quickstart.py)（只用标准库，会顺带演示翻页）：`python3 examples/quickstart.py`。

返回统一的结构：列表在 `data.items`，下一页用 `page.next_cursor` 传回 `cursor`，这次扣了多少钱在 `meta.billing.amount`，平台不提供的字段为 null 并在 `meta.unavailable` 里写明原因。

## 让 AI 直接用（MCP）

Claude Code：

```bash
claude mcp add --transport http --scope user okdata https://api.okdata.online/mcp --header "Authorization: Bearer $OKDATA_API_KEY"
```

Cursor、Cherry Studio、WorkBuddy 等支持远程 MCP 的客户端：

```json
{
  "mcpServers": {
    "okdata": {
      "url": "https://api.okdata.online/mcp",
      "headers": {
        "Authorization": "Bearer okd_..."
      }
    }
  }
}
```

接好以后直接用中文说需求，比如「快手视频搜索：露营装备」。AI 会先用免费的 `find`、`describe` 找接口和看价格，再用 `call` 取数据。

## 常见问题

**要登录平台账号吗？** 不用。数据是公开内容，okdata 在服务端获取，你的账号完全不参与。

**怎么收费？** 每个接口单独定价，表里写的就是一次调用的价格；按条计费的接口按实际返回条数算。新账号送 $0.1 试用额度，之后 $10 起充，余额不过期，没有订阅。价格页：https://okdata.online/zh/pricing/ ，计费说明：https://okdata.online/zh/docs/billing/

**和官方开放平台有什么区别？** 官方开放平台只能读授权给你的账号自己的数据，而且要企业资质和审核；这里是平台上任何人都能看到的公开数据，比如搜索结果、别人的笔记或视频、评论、热榜。

**能翻页吗？** 列表接口每页一次调用，把 `page.next_cursor` 原样传回 `cursor` 就是下一页，24 小时内有效。

**出错了怎么办？** 每个错误都有固定的 `code`、中文说明 `message_zh`，以及会不会扣费（`charged`）和要不要重试（`retry`）。全部错误码：https://okdata.online/zh/docs/errors/

## English

Examples for the okdata Kuaishou API: 8 live endpoints behind one API key, pay per call from $0.002, no platform login or cookies. Sign up at https://okdata.online/signup/?src=gh-kuaishou-api, docs at https://okdata.online/apis/, MCP at https://okdata.online/mcp/.

## License

MIT. The data comes from the okdata API; its terms apply to API use.
