# 阿里叔叔面板自动续期 · 公开工作流（4）

这个仓库**只有 workflow**（`.github/workflows/renew.yml`），没有账号代码。
代码在私有库（`vars.CODE_REPO`），运行时 checkout 到 `app/` 再执行。

- 账号：4
- 定时：15 2 * * * UTC（北京时间 10:15）
- 状态：私有库 `state/acc4.json`（cookie + 浏览器指纹，跑完自动回写）
- 代理：本库专属 `PROXY_NODE`

## 要配的东西

Variables：`CODE_REPO`（必填，`YYluck06/alishushu-renew-core`）、`INSTANCE`、`RENEW_MONTHS`、`SKIP_IF_DAYS_GT`、`MAX_COST`
Secrets：`CODE_REPO_TOKEN`、`ACC_EMAIL`、`ACC_PASS`、`ACC_SEED`、`PROXY_NODE`、`TG_TOKEN`、`TG_CHAT`

`PROXY_NODE` 支持：

```
socks5://host:port                    # socks5 无认证
socks5://user:pass@host:port          # socks5 带用户名密码
hy2://pass@host:443?sni=xx&insecure=1 # hysteria2
vmess://… / vless://… / trojan://… / tuic://… / ss://…
```

留空 = 直连（不推荐）。

## 手动触发

Actions → 阿里叔叔续期 · 账号4 → Run workflow：
- `dry`：只报价不确认
- `selftest`：只跑代理隔离自测（4 个假节点，不碰账号）
- `discover`：只跑校准，把控制台结构存成 artifact
- `instance`：只续名称/ID 含该字符串的实例