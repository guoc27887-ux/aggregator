# Aggregator 部署说明（fork）

Fork：https://github.com/guoc27887-ux/aggregator（上游 wzdnzd/aggregator，commit 5d72831）
目标：每 6 小时自动抓取公开免费节点，生成 Clash / V2Ray / Sing-box 订阅，保存到 Gist。

## 定时

- Refresh（主流程）：每 6 小时 `0 */6 * * *` —— 刷新已有订阅、重新检测、重新转换、上传 Gist；Gist 为空时首次运行会自动发现新机场。
- Collect（发现新机场）：每周日 00:00 —— 爬取新机场并注册免费账号。
- Process：已停用定时（需要 SUBSCRIBE_CONF 自定义配置，本部署不用），仅手动触发。

## 订阅地址（Gist raw）

- Clash：https://gist.githubusercontent.com/guoc27887-ux/6643ad96b751412c39c44ea083d2c9de/raw/clash.yaml
- V2Ray：https://gist.githubusercontent.com/guoc27887-ux/6643ad96b751412c39c44ea083d2c9de/raw/v2ray.txt
- Sing-box：https://gist.githubusercontent.com/guoc27887-ux/6643ad96b751412c39c44ea083d2c9de/raw/singbox.json
- 订阅源列表：https://gist.githubusercontent.com/guoc27887-ux/6643ad96b751412c39c44ea083d2c9de/raw/subscribes.txt
- Gist 页面：https://gist.github.com/guoc27887-ux/6643ad96b751412c39c44ea083d2c9de

## 本 fork 相对上游的改动

1. Refresh 每 6 小时（上游每 2 小时）；Collect 每周日发现新机场不变。
2. Process 工作流停用定时（需要 SUBSCRIBE_CONF 自定义配置，本部署不用），仅手动触发。
3. collect.py 的 v2ray 修复：上游自带的 subconverter v0.9.9 对 target "v2ray" 只输出 VMess/SS 链接，VLESS/Trojan 节点会被静默丢掉，导致 v2ray.txt 为空（2026-11-12 本地实测：2 个 VLESS 节点 → v2ray.txt 0 字节）。本 fork 改为用 target "mixed"（list=false 时直接输出标准 base64 V2Ray 订阅，含全部协议），文件名仍是 v2ray.txt。验证：2 个 VLESS 节点 → v2ray.txt 680 字节 base64，单次解码即 2 条 vless 链接；clash.yaml（2 节点）与 singbox.json（17 outbounds）同期验证通过；两节点经 sing-box 实测 generate_204 均为 204。

## 需要的 Secrets（仓库 Settings → Secrets and variables → Actions）

- GIST_LINK：已设置，`guoc27887-ux/6643ad96b751412c39c44ea083d2c9de`
- **GIST_PAT：还没设置，需要用户创建** —— GitHub 个人访问令牌，只需要 gist 权限。步骤：GitHub 头像 → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token → 只勾选 `gist` → 复制 token → 回到本仓库 Settings → Secrets and variables → Actions → New repository secret，名字填 `GIST_PAT`，值粘贴 token。Token 只存在 Secrets 里，不要贴到聊天、文件或仓库里。
- CUSTOMIZE_LINK（可选）：自己维护的机场列表 URL，每行 `域名 [优惠码] [邀请码]`。
- 变量 ENABLE_SPECIAL_PROTOCOLS（可选）：开启 vless / hysteria2 等特殊协议。

## 失败时保留上次结果（项目自带机制，不用改代码）

- 抓不到任何可用节点时，collect.py 在上传前就退出（`cannot found any valid config` / `cannot fetch any proxy`），Gist 不动。
- 转换全部失败时 exit 1，同样不上传。
- 上传走 Gist PATCH，只更新本次成功生成的文件，没生成的文件保持旧内容。
- 工作流开头的 Check 步骤在 GIST_PAT / GIST_LINK 为空时直接失败，不会动 Gist。

## Shadowrocket 导入（手机上的步骤）

1. 打开 Shadowrocket → 配置 → 右上角 + → 添加订阅。
2. 粘贴 Clash 订阅地址（上面 clash.yaml 的 raw 链接），保存。
3. 回到首页，下拉刷新订阅，选一个节点，打开开关。
4. V2Ray 链接（v2ray.txt）也可直接作为订阅导入；singbox.json 给 Sing-box 客户端用。

## 手动触发 / 查看运行

- 仓库 Actions 标签页 → Refresh → Run workflow。
- 注意：GitHub Actions 定时任务在 fork 上需要在 Settings → Actions → General 里允许运行。

## 提醒

- 公开免费池大多到手即死：2026-11-11 本地实测抓 444 个机场、可用 0 个。订阅为空不代表流程坏了，先看 Actions 运行日志。
- 不要把 GIST_PAT 贴到任何公开地方。
