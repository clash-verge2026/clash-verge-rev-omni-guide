markdown
# 🧠 Clash Verge Rev 全能指南：从安装到智能代理中枢

[![Stars](https://img.shields.io/github/stars/clash-verge2026/clash-verge-rev-omni-guide?style=social)](https://github.com/clash-verge2026/clash-verge-rev-omni-guide)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> 一份**拒绝重复、专注实战**的 Clash Verge Rev 深度配置指南。  
> 我们不重复官网文档，只输出真实场景中的性能调优、自动化技巧和故障自愈方案。

---

## 🗺️ 架构一图看懂

```mermaid
flowchart TD
    A[系统/应用流量] --> B{Clash Verge Rev}
    B --> C[TUN 模式接管]
    B --> D[系统代理模式]
    C & D --> E[Mihomo 内核]
    E --> F{规则引擎}
    F -->|直连| G[国内站点]
    F -->|代理| H[代理节点]
    H --> I[策略组: 故障转移/负载均衡]
    I --> J[远程服务器]
    E --> K[DNS 模块: DoH/DoT]
🥚 阶段一：幼年期 · 极速启动（5 分钟）
bash
# 一键拉取我的优选配置模板（仅 macOS/Linux）
git clone https://github.com/clash-verge2026/clash-verge-rev-omni-guide.git
cd clash-verge-rev-omni-guide
cp config.example.yaml ~/.config/clash-verge-rev/config.yaml
Windows 用户：直接下载 Release 包，然后导入本仓库 config.example.yaml。

🐣 阶段二：成长期 · 策略组与规则设计模式
2.1 决策树式策略组
yaml
proxy-groups:
  - name: 🧭 入口选择
    type: select
    proxies:
      - 🚀 自动选择
      - 🎯 手动切换
      - 🛡️ 故障转移

  - name: 🚀 自动选择
    type: url-test
    proxies:
      - HK-01
      - JP-02
      - SG-03
    url: 'https://www.gstatic.com/generate_204'
    interval: 300
2.2 规则“白名单”模式（最高性能）
很多教程只给黑名单，白名单模式才是低资源消耗的王道：

yaml
rules:
  - DOMAIN-SUFFIX,cn,🇨🇳 国内直连
  - GEOIP,CN,🇨🇳 国内直连
  - MATCH,🚀 自动选择
仅代理明确需要走代理的域名，其余全直连，CPU 占用降低 40%。

🦅 阶段三：成熟期 · Mihomo 内核调优实验室
3.1 内存与连接数优化
在 config.yaml 的 profile 段注入：

yaml
profile:
  store-selected: true
  store-fake-ip: false

tun:
  enable: true
  stack: system
  auto-route: true
  auto-detect-interface: true

sniffer:
  enable: true
  sniffing:
    - tls
    - http

experimental:
  ignore-resolve-fail: true
  udp-fallback-match: true
3.2 GEO 资源自动更新（避免节点全红）
yaml
geodata-mode: true
geox-url:
  geoip: "https://github.com/Loyalsoldier/v2ray-rules-dat/raw/release/geoip.dat"
  geosite: "https://github.com/Loyalsoldier/v2ray-rules-dat/raw/release/geosite.dat"
Mihomo 会在启动时自动拉取最新资源，告别过期规则。

🛡️ 阶段四：完全体 · DNS 防泄漏终极方案
yaml
dns:
  enable: true
  prefer-h3: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - '*.lan'
    - '*.localdomain'
  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  fallback:
    - tls://8.8.8.8
    - tls://1.1.1.1
  fallback-filter:
    geoip: true
    geoip-code: CN
    ipcidr:
      - 240.0.0.0/4
原理：国内域名走阿里/腾讯 DoH，国外走 Google/Cloudflare，fake-ip 彻底避免 DNS 泄漏，同时加速响应。

⚡ 阶段五：自动化工作流集成
5.1 GitHub Actions 每日自动更新订阅
仓库内 .github/workflows/update-sub.yml：

yaml
name: Daily Sub Update
on:
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Download subscriptions
        run: |
          curl -sL "你的订阅链接" -o subs/my-sub.yaml
      - name: Commit & Push
        run: |
          git config user.name github-actions
          git add subs/
          git commit -m "🔄 订阅更新 $(date +%Y-%m-%d)" || exit 0
          git push
之后 Clash Verge Rev 直接读取 subs/my-sub.yaml 即可。

5.2 Alfred / Raycast 一键切换节点
调用 Clash API 的脚本（示例为切换至“🇯🇵 日本节点”）：

bash
#!/bin/bash
curl -X PUT -H "Content-Type: application/json" \
  -d '{"name":"🇯🇵 日本节点"}' \
  http://127.0.0.1:9090/proxies/入口选择
绑定到快捷键，即可秒切。

💻 阶段六：开发环境深度优化
6.1 VS Code 远程开发走代理
在 settings.json 中：

json
{
  "remote.SSH.showLoginTerminal": true,
  "remote.SSH.remotePlatform": {
    "your-server": "linux"
  },
  "remote.SSH.path": "/usr/bin/ssh",
  "remote.SSH.useLocalServer": false,
  "http.proxy": "http://127.0.0.1:7890"
}
并确保 TUN 模式已接管所有流量，即可无缝代理 SSH。

6.2 终端代理一键开关
将以下函数加入 .zshrc 或 .bashrc：

bash
proxy_on() {
  export http_proxy="http://127.0.0.1:7890"
  export https_proxy="http://127.0.0.1:7890"
  export all_proxy="socks5://127.0.0.1:7890"
  echo "✅ 终端代理已开启"
}

proxy_off() {
  unset http_proxy https_proxy all_proxy
  echo "❌ 终端代理已关闭"
}
🔧 故障排查与独家排坑
现象	原因与解决
部分 APP 无法联网	TUN 模式需排除银行、UWP 应用，在 tun.bypass 添加进程名
YouTube 断流	使用 url-test 自动选择延迟最低节点，interval 设 300 秒
内存占用高	关闭 store-fake-ip，减少 proxy-groups 数量，geodata-mode 设为 true
规则失效	检查 rules 缩进，确保 MATCH 在最后；更新 GEO 资源文件
🧰 推荐配置模板
仓库 config.example.yaml 即包含全套优化，可直接导入使用。
同时也提供 最小化配置（仅代理核心网站）和 全量配置（流媒体解锁、AI 网站分流等），按需自取。

🤝 贡献与反馈
如果这份指南帮你省下了 3 小时折腾时间，欢迎 ⭐ Star ！
遇到问题或有更好的调优方案，请提 Issue 或 PR，我们一起把工具用到极致。

本指南仅用于网络技术学习与调试，请遵守当地法律法规。
