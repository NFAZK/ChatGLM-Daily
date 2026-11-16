# 部署说明（GitHub Actions 每日签到）

这个仓库是 [L0NE-6/ChatGLM-Daily](https://github.com/L0NE-6/ChatGLM-Daily)（MIT）的 **Fork**，签到逻辑、签名算法、令牌续期、任务结算全部来自原作者，未做改写。
本 Fork 只做了一件事：**把原本被注释掉的定时触发打开**，让它每天自动跑。原作者的 `README.md`、`NOTICE.md` 的使用边界请一并遵守。

## 已验证（2026-11-16 实跑）

| 场景 | 结果 |
|---|---|
| 本机完整运行 | 签到 + 每日任务 + 新手任务全部结算，余额 1200 → 6700 |
| Actions 手动触发 | 全步骤 `success`，云端登录态可用，**GitHub 海外 IP 未被拦截** |
| 令牌轮换 | 同一枚 refresh token 连续使用两次均成功 → 旧 token 不会因轮换失效，静态 Secret 可长期使用 |
| 幂等 | 同日重复运行时返回"今日登录积分已领取 / 任务 0 个"，不会报错也不会重复发放 |

refresh token 有效期约到 2027 年 4 月，到期前重新取一次即可。

## 当前状态

| 项 | 值 |
|---|---|
| 定时 | `.github/workflows/daily.yml` 中 `cron: "10 0 * * *"` |
| 含义 | UTC 0:10 = **北京时间 08:10**，每天一次 |
| 手动触发 | Actions → 左侧 `Zhipu Daily Check-in` → Run workflow |

改时间就改那一行 cron（**cron 用 UTC，北京时间 = UTC+8**，所以北京时间 8 点写 `0 0`，22 点写 `14 0`）。

## 还差两步（都要你自己做，涉及账号凭据）

### 1. 取登录态

1. 浏览器打开 https://chatglm.cn 并登录
2. `F12` → **Application / 应用** → 左侧 **Cookies** → `https://chatglm.cn`
3. 复制这两条的 Value：
   - `chatglm_token` —— access token
   - `chatglm_refresh_token` —— refresh token（有它才能自动续期）

> 有 `chatglm_device_id` 就一起填；没有留空，脚本会从 JWT 的 `device_id` 声明里取。

### 2. 填 Secrets

本仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**：

| Name | Value |
|---|---|
| `CHATGLM_TOKEN` | access token |
| `CHATGLM_REFRESH_TOKEN` | refresh token |
| `CHATGLM_DEVICE_ID` | 设备 UUID（可选） |

**多账号**用一个 Secret 代替上面三个：

```
Name:  CHATGLM_ACCOUNTS
Value: 主账号|ACCESS_TOKEN|REFRESH_TOKEN|DEVICE_UUID
       小号|ACCESS_TOKEN|REFRESH_TOKEN|DEVICE_UUID
```

推送通知（可选）：`PLUSPLUS_TOKEN` / `BARK_URL` / `WECOM_WEBHOOK` / `DINGTALK_WEBHOOK`。

⚠️ 这些等同于你账号的登录凭据。只放 Secret，不要写进仓库里的任何文件，也不要贴到 Issue / 聊天里。

## 填完之后

Actions → `Zhipu Daily Check-in` → **Run workflow**，等 1~2 分钟看日志：

- 能看到每个账号的签到结果、任务完成数、积分余额 → 通了
- 报访客账号被跳过 → 用正式账号的令牌
- 报 401 → 令牌过期，重新取一次，更新 Secret

跑通后什么都不用管，每天 08:10 自动执行。

## 常见问题

| 现象 | 原因 | 处理 |
|---|---|---|
| 定时不触发 | 仓库 60 天没有提交，Actions 定时会被暂停 | 手动 Run workflow 一次即可恢复 |
| 令牌失效 | refresh token 过期（约 2027-04）或被服务端作废 | 重新取一次令牌，更新 Secret |
| 签到成功但积分没涨 | 活动空窗期，或当日已领 | 中性结果，脚本不会伪造成功 |
| 想同步上游更新 | 本仓库是 Fork | GitHub 页面上点 **Sync fork**，注意别把 `daily.yml` 的定时改回注释 |

## 边界与风险

- 只做签到与任务领取，不修改服务端数据
- 依赖上游的 `X-Sign` 客户端签名算法，**智谱一改版就失效**，要等作者更新
- 自动化行为可能不符合智谱服务条款，风控风险自担；建议单账号、每天一次，别调高频
- Actions 出口是海外共享 IP，比家宽更容易被判定异常；在意的话放国内服务器/青龙更稳

## 许可

沿用上游 MIT，见 [LICENSE](LICENSE)；使用边界见 [NOTICE.md](NOTICE.md)（官方版本永久免费，禁止付费包装、搬运引流、冒充原创）。
