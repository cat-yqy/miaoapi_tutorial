# 详细配置：端口、TUN、DNS、规则与扩展

这一页面向已经能正常上网、想进一步把 Clash Verge Rev 配稳的人。先改最少的项，确认每一步生效，再继续。

!!! tip "先备份"
    改配置前用 WebDAV 或手动复制配置目录备份。规则、DNS、TUN 同时改，出问题会很难定位。

## 1. 网络与端口

常见设置：

| 设置 | 说明 | 建议 |
| --- | --- | --- |
| 混合端口 / Mixed Port | HTTP + SOCKS 共用端口 | 默认 `7897`；被占用时换一个 |
| 允许局域网 / Allow LAN | 让同网段设备使用代理 | 默认关闭；需要时再开 |
| 绑定地址 | 监听 `127.0.0.1` 还是 `0.0.0.0` | 只给本机用就保持 localhost |
| IPv6 | 是否启用 IPv6 | 网络支持且需要时再开 |
| 统一延迟 / Unified Delay | 测速使用统一握手方式 | 一般开启，延迟更可比 |
| 日志等级 | 记录多少日志 | 日常 `warning`，排错临时 `info/debug` |

### 端口被占用

默认端口 `7897` 被其他程序占用时，系统代理会失败。

```bash
# macOS / Linux
lsof -i :7897

# Windows PowerShell
netstat -ano | findstr :7897
```

换端口后，系统代理会自动使用新端口；手动配置过代理的程序要一起改。

### 允许局域网

开启后，同一局域网的设备可以把代理指向：

```text
http://<你的电脑 IP>:7897
```

注意：

- 电脑防火墙要放行该端口；
- 不要把代理端口直接暴露到公网；
- 公共 Wi-Fi 下不要开启允许局域网；
- 需要认证时，考虑在上一层网络/网关做访问控制，而不是只依赖端口。

## 2. 系统代理与守卫

### 系统代理

打开后，Clash Verge Rev 会修改操作系统的代理设置。适合浏览器和遵守系统代理的桌面应用。

限制：

- 不处理 UDP；
- 不遵守系统代理的程序无效；
- 某些应用（例如 App Store 版 Telegram）需要单独设置代理，或改用 TUN。

### 系统代理守卫

“守卫”用于防止其他程序修改系统代理设置。开启后，其他软件改动代理可能被拦截或恢复。

如果你使用公司 VPN、抓包工具、其他代理客户端，守卫可能会和它们冲突；出现“代理设置被改回去”时先检查这里。

## 3. TUN 模式

TUN 通过虚拟网卡接管流量，适合命令行、游戏和 UDP。

常见选项：

| 选项 | 作用 |
| --- | --- |
| 栈 / Stack | `gVisor`（默认）、`system`、`mixed`；不同系统兼容性不同 |
| 自动路由 / Auto Route | 自动添加路由，把流量导向 TUN |
| 严格路由 / Strict Route | 防止流量绕过 TUN；排错时可临时关闭 |
| DNS 劫持 / DNS Hijack | 把 DNS 请求导入 Mihomo |
| MTU | 虚拟网卡 MTU，默认常见 1500 |
| 自动检测接口 | 自动选择物理网卡 |

### 开启 TUN 的前置条件

1. 服务模式安装成功；
2. 没有其他 VPN/加速器占用虚拟网卡；
3. 防火墙允许 `verge-mihomo` / `verge-mihomo-alpha`；
4. 系统没有异常的额外网卡/网段冲突。

### 常见冲突

- 公司 VPN、ZeroTier、Tailscale、游戏加速器、Watt Toolkit；
- 虚拟机/容器网络；
- 多张网卡、多个默认路由；
- 某些安全软件的网络过滤驱动。

TUN 下网络异常时，先关闭其他 VPN/加速器，再用“直连”模式确认基础网络正常。

## 4. DNS

DNS 是 Clash Verge Rev 最容易配出问题的一块。**能正常用就不要乱改。**

常见字段：

```yaml
dns:
  enable: true
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  nameserver-policy:
    "+.corp.example.com": "10.0.0.1"
  fake-ip-filter:
    - "+.lan"
    - "+.local"
    - "time.windows.com"
```

### fake-ip

`fake-ip` 会给域名返回一个虚拟 IP，客户端拿到虚拟 IP 后由 Mihomo 按域名分流。它通常能提升分流准确性，但可能影响：

- 依赖真实 DNS 结果的程序；
- 局域网设备发现；
- 某些游戏/语音应用；
- 需要直接解析域名的脚本。

遇到问题可以把特定域名加入 `fake-ip-filter`，或改用 `redir-host`（不推荐作为默认）。

### 内网域名解析失败

公司内网域名通常无法通过公共 DNS 解析。需要给内网域名指定内网 DNS：

```yaml
dns:
  nameserver-policy:
    "+.corp.example.com": "10.0.0.1"
    "+.internal.example.com": "10.0.0.1"
```

注意：从 v2.5.5 起，扩展配置里写出的 `nameserver-policy` 会**整体替换**订阅原有值。要保留订阅条目，就把它们一起写上，或改用扩展脚本追加。

### DNS 覆写

设置里的“DNS 覆写”优先级高于订阅和扩展。开启后，覆写页填写的 `dns` 字段和 `hosts` 会替换订阅的对应字段；关闭时应用不改动订阅的 DNS。

如果你发现“改了扩展配置但 DNS 没变”，先检查 DNS 覆写是否开启。

## 5. 规则

规则按从上到下的顺序匹配，命中第一条就停止。

常见类型：

```yaml
rules:
  - DOMAIN,example.com,PROXY
  - DOMAIN-SUFFIX,example.org,PROXY
  - DOMAIN-KEYWORD,ads,REJECT
  - GEOSITE,cn,DIRECT
  - GEOIP,CN,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve
  - DST-PORT,22,DIRECT
  - PROCESS-NAME,Telegram.exe,PROXY
  - AND,((DOMAIN-SUFFIX,example.com),(GEOIP,!CN)),PROXY
  - MATCH,PROXY
```

策略目标：

- `DIRECT`：直连；
- `REJECT`：阻断；
- 代理组名称：例如 `手动选择`、`自动选择`、`AI`。

### 前置规则与后置规则

在订阅卡片的“编辑规则”里：

- **前置规则（prepend）**：优先于订阅规则，推荐放自定义规则；
- **后置规则（append）**：在订阅规则之后，只有前面都没命中才生效。

### 常用示例

**国内直连：**

```yaml
- GEOSITE,cn,DIRECT
- GEOIP,CN,DIRECT
- MATCH,PROXY
```

**广告拦截：**

```yaml
- GEOSITE,category-ads-all,REJECT
- DOMAIN-KEYWORD,ads,REJECT
```

**应用分流：**

```yaml
- PROCESS-NAME,Telegram.exe,PROXY
- PROCESS-NAME,WeChat.exe,DIRECT
```

`PROCESS-NAME` 依赖进程匹配能力；某些系统/权限下可能拿不到进程名，规则会失效。

### 排错顺序

规则不生效时：

1. 看“连接”页实际命中的规则；
2. 确认自定义规则在前置；
3. 确认订阅更新没有覆盖；
4. 确认域名/IP 与规则类型匹配；
5. 确认策略组名称没有写错；
6. 确认 `MATCH` 在最后。

## 6. 代理组

常见类型：

| 类型 | 行为 |
| --- | --- |
| `select` | 手动选择节点 |
| `url-test` | 自动选择延迟最低 |
| `fallback` | 按顺序故障转移 |
| `load-balance` | 负载均衡 |
| `relay` | 链式/中继（取决于配置） |

建议：

- 日常主入口用 `select`，自己掌控；
- 需要自动选优再加 `url-test`；
- 不要把全部节点塞进一个巨大的 `url-test`，测速和切换会很慢；
- 给不同用途建独立组（AI、流媒体、下载、游戏），规则更清晰。

## 7. 链式代理

v2.4.5+ 支持链式代理，适合“机场入口 + 家庭宽带出口”等场景。

步骤：

1. 右键订阅 → **编辑节点**；
2. 添加节点，例如：

```text
http://user:pass@192.168.1.100:8080
socks5://user:pass@home.example.com:1080
```

3. 按顺序点击节点加入代理链：
   - 入口节点在最上面；
   - 中间节点可选；
   - 出口节点在最下面；
4. 在规则或代理组里使用这条链。

注意：

- 延迟会累加；
- 速度取决于最慢的一环；
- 任何一环失效，整条链失效；
- 认证信息中的特殊字符要 URL 编码；
- 编辑节点不会随订阅更新丢失。

## 8. 扩展配置与扩展脚本

执行顺序：

```text
订阅原文 → 应用设置 → 全局扩展配置 → 全局扩展脚本
        → 订阅扩展配置 → 订阅扩展脚本 → 应用设置回写
```

### 扩展配置

用 YAML 覆写/合并字段：

```yaml
dns:
  enable: true
  ipv6: false
tcp-concurrent: true
```

规则：

- 列表和普通值整体替换；
- 嵌套对象逐项合并；
- `dns` 下写出的字段和顶层 `hosts` 整体替换；
- 设置页接管的字段（端口、模式、TUN 等）最终以应用设置为准。

### 扩展脚本

用 JavaScript 做更灵活的处理：

```js
function main(config) {
  config.dns = config.dns || {};
  config.dns["nameserver-policy"] = config.dns["nameserver-policy"] || {};
  config.dns["nameserver-policy"]["+.corp.example.com"] = "10.0.0.1";
  return config;
}
```

适合：

- 动态追加规则；
- 按条件改 DNS；
- 批量调整节点；
- 处理订阅里的特殊字段。

不要写会阻塞或无限循环的脚本；每次配置加载都会执行。

## 9. 多订阅合并

如果有多份订阅，可以用 `proxy-providers` 合并，再用统一的代理组和规则管理。

示例：

```yaml
proxy-providers:
  provider1:
    type: http
    path: ./proxy_provider/p1.yaml
    url: https://example.com/sub1
    interval: 21600
    override:
      additional-prefix: "P1 |"

  provider2:
    type: http
    path: ./proxy_provider/p2.yaml
    url: https://example.com/sub2
    interval: 21600
    override:
      additional-prefix: "P2 |"
```

要点：

- `path` 是本地缓存文件；
- `interval` 是更新间隔（秒）；
- `additional-prefix` 给节点加前缀，避免同名；
- 同名节点可能导致代理组显示 `ambiguous`；
- 规则里引用的组名必须存在。

更完整的合并示例见[官方多订阅配置](https://clash-verge-rev.github.io/guide/config.html)。

## 10. WebDAV 备份

设置里可以配置 WebDAV，用于备份/同步配置。

建议：

- 使用 HTTPS；
- 使用独立应用密码，不要用主账号密码；
- 备份包含订阅链接/节点信息，按敏感数据对待；
- 换机前先手动导出一次，确认能恢复。

## 11. 外观与启动

- 主题、颜色、托盘图标、CSS 注入：按个人喜好；
- 开机自启：需要常驻代理时开启；
- 静默启动：启动时不显示主窗口；
- 托盘菜单：快速切换系统代理/TUN/模式/节点；
- 自动检查更新：建议开启；
- 日志等级：日常 `warning`，排错临时调高。

## 配置检查清单

- [ ] 端口没有被占用
- [ ] 只给本机用时不开 Allow LAN
- [ ] TUN 只在需要时开
- [ ] 服务模式正常
- [ ] DNS 没有盲目改
- [ ] 自定义规则放前置
- [ ] 规则目标组名存在
- [ ] 多订阅节点名不冲突
- [ ] 已备份配置

## 官方参考

- [名词解释](https://clash-verge-rev.github.io/guide/term.html)
- [自定义规则](https://clash-verge-rev.github.io/guide/rules.html)
- [链式代理](https://clash-verge-rev.github.io/guide/proxy_chain.html)
- [扩展配置与脚本](https://clash-verge-rev.github.io/guide/extend.html)
- [多订阅合并](https://clash-verge-rev.github.io/guide/config.html)
- [Mihomo 配置文档](https://wiki.metacubex.one/config/)
