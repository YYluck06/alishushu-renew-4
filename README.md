# 阿里叔叔面板 · 自动延长 72 小时（账号4）

这个仓库**只放一个 workflow**，账号代码在私有库 `YYluck06/alishushu-renew-core`，
运行时用 `CODE_REPO_TOKEN` 检出到 `alishushu-core/` 再跑。公开库里没有任何账号代码。

- 定时：`15 2 * * *`（UTC）= 北京时间 **10:15**
- 动作：每天给该账号下**每台实例**点一次「延长 72 小时」——免费、次数不限
- 触发线：剩余延长期 < `EXTEND_KEEP_HOURS`（默认 48 小时）才点下一次，所以**实际两天点一次**
- 登录：指纹 + 会话存在私有库的 `state/acc4.json`，登录一次长期复用，不重复输密码
- 报告：每轮都发一条 Telegram（跳过也发，不然分不清「没到期」还是「workflow 挂了」）

## 要配的东西

Settings → Secrets and variables → Actions

**Secrets**

| 名字 | 必填 | 说明 |
| --- | --- | --- |
| `CODE_REPO_TOKEN` | ✅ | 读私有库的 token，权限 Contents: **Read and write**（要回写会话状态） |
| `ACC_EMAIL` / `ACC_PASS` | ✅ | 账号4 的账号（用户名或邮箱都行）与密码 |
| `ACC_SEED` | | 指纹种子，留空用账号派生，同一账号永远同一指纹 |
| `PROXY_NODE` | | 本库**专属**出口，见下表；留空 = 直连（不推荐） |
| `TG_TOKEN` / `TG_CHAT` | | Telegram 报告，四个库可共用同一个 bot |

**Variables**

| 名字 | 默认 | 说明 |
| --- | --- | --- |
| `CODE_REPO` | — | `YYluck06/alishushu-renew-core`（必填，不配就拉不到私有库） |
| `ACTION` | `delay` | `delay` = 免费延长 72 小时；`renew` = 花钱续费 |
| `EXTEND_CLICKS` | `1` | 每台实例点几次延长 |
| `EXTEND_KEEP_HOURS` | `48` | 剩余延长期低于这么多小时才点下一次 |
| `INSTANCE` | — | 只处理名称含该子串的实例 |
| `RENEW_MONTHS` | `1` | 仅 `ACTION=renew` |
| `SKIP_IF_DAYS_GT` | `30` | 仅 `ACTION=renew` |
| `MAX_COST` | `0` | 仅 `ACTION=renew`，0 = 只要要钱就中止 |

## `PROXY_NODE` 支持的协议

| 写法 | 说明 |
| --- | --- |
| `socks5://host:port` | socks5，无用户名密码 |
| `socks5://user:pass@host:port` | socks5，带用户名密码 |
| `hy2://pass@host:443?sni=x&insecure=1` | hysteria2（也认 `hysteria2://`） |
| `tuic://…` `trojan://…` `vmess://…` `vless://…（reality）` `ss://…` | 全部由 sing-box 原生支持，不需要额外装 hysteria/tuic |

节点链接、主机、出口 IP 在日志里都会打成 `***`（`::add-mask::`）。

## 手动跑

Actions → 本 workflow → Run workflow：

| 输入 | 说明 |
| --- | --- |
| `phase` | `auto`（默认）/ `delay` 只延长 / `renew` 花钱续费 / `selftest` 代理隔离自测 / `discover` 扒站点结构 |
| `force` | 勾上 = 忽略「到期窗口 / 触发线」强制点（排障用） |
| `dry_run` | 勾上 = 只体检不动手 |
| `clicks` | 每台点几次，留空用 `EXTEND_CLICKS` |

## 两个按钮别搞混

| 按钮 | 花不花钱 | 接口 | 本 workflow |
| --- | --- | --- | --- |
| **续费** | 💰 扣钱包 | `POST /console/subscriptions/{id}/renew` | 只有 `ACTION=renew` 才走，带余额/价格上限门禁 |
| **延长 72 小时** | 🆓 免费、次数不限 | `POST /console/subscriptions/{id}/delay-stop` | **默认就是这个** |

> 延长的时间算欠费，续费从原到期时间起算；到期关机超过 3 天的实例会被删除，所以 cron 别断太久。

## 加一个账号

复制 `renew.yml` 改 5 处（文件头用 ①②③④⑤ 标了）：`name`、`cron`、`ACC_EMAIL`（改 `_2` 后缀）、
`ACC_PASS`（改 `_2` 后缀）、`STATE_FILE`/`TIME_DIR`（改 `acc2` / `acc/2`）。
或者直接新建一个公开库（本项目的做法：4 个账号 = 4 个库，各一条出口、各自的 cron）。
