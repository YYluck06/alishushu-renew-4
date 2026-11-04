# 阿里叔叔面板自动延长 · 公开工作流（账号4）

这个仓库**只有 workflow**（`.github/workflows/renew.yml`），没有账号代码。
代码在私有库（`vars.CODE_REPO`），运行时 checkout 到 `app/` 再执行。

- 账号：4
- 定时：`15 2 * * *` UTC（北京时间 10:15）
- 每日动作：登录（会话长期复用）→ 给**每台实例**点「延长 72 小时」（免费、不限次数）→ 回写指纹/会话 → TG 通知
- 状态：私有库 `state/acc4.json`（cookie + 浏览器指纹，跑完自动回写）
- 代理：本库专属 `PROXY_NODE`

## 要配的东西

Variables：`CODE_REPO`（必填，`YYluck06/alishushu-renew-core`）、`ACTION`（默认 `delay`；改成 `renew` 才是花钱续费）、`EXTEND_CLICKS`（每台点几次，默认 1）、`INSTANCE`、`RENEW_MONTHS`、`SKIP_IF_DAYS_GT`、`MAX_COST`
Secrets：`CODE_REPO_TOKEN`、`ACC_EMAIL`、`ACC_PASS`、`ACC_SEED`、`PROXY_NODE`、`TG_TOKEN`、`TG_CHAT`

## 手动触发

Actions → 「阿里叔叔续期 · 账号4」→ Run workflow：

- 什么都不勾 = 真跑（点「延长 72 小时」）
- `dry` = 演练，只读不点
- `selftest` = 只跑代理隔离自测（本机假节点，不碰账号）
- `discover` = 只跑校准，把控制台结构存 artifact
- `clicks` = 这次每台点几次（留空用 `EXTEND_CLICKS`）

## 「延长 72 小时」和「续费」的区别

| 按钮 | 花钱 | 接口 | 说明 |
|---|---|---|---|
| 延长 72 小时 | 🆓 免费、次数不限 | `POST /console/subscriptions/{id}/delay-stop` | 每点一次从现在起再运行 72 小时；到期前 3 天内才给点 |
| 续费 | 💰 扣钱包 | `POST /console/subscriptions/{id}/renew` | 只有 `ACTION=renew` 才走，带余额/价格上限门禁 |

`PROXY_NODE` 支持：

```
socks5://host:port                    # socks5 无认证
socks5://user:pass@host:port          # socks5 带用户名密码
hy2://pass@host:443?sni=xx&insecure=1 # hysteria2
vmess://… / vless://… / trojan://… / tuic://… / ss://…
```

留空 = 直连（不推荐）。