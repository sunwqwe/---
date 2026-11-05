# 🛡️ CDN 绕过技术：寻找真实 IP 实战手册

> 绕过 CDN 找真实 IP 的完整技术手册，适用于**授权渗透测试、红蓝攻防与安全自查**。

⚠️ **免责声明**：本手册仅用于合法授权的安全测试与防御自查。未经授权对目标进行探测可能违反《网络安全法》《刑法》第 285/286 条及相关法律法规，使用者需自行承担一切法律责任。

---

## 📖 目录

- [核心思路](#核心思路)
- [方法总览（按命中率排序）](#方法总览按命中率排序)
- [方法一：历史 DNS 解析（首选）](#方法一历史-dns-解析首选)
- [方法二：子域名旁路](#方法二子域名旁路)
- [方法三：证书透明度日志](#方法三证书透明度日志)
- [方法四：网络空间测绘平台](#方法四网络空间测绘平台fofashodan-语法直接可用)
- [方法五：邮件头溯源](#方法五邮件头溯源)
- [方法六：源码泄露与配置文件](#方法六源码泄露与配置文件)
- [方法七：SSRF 与错误页面](#方法七ssrf-与错误页面)
- [方法八：全网扫描 + Host 头碰撞（终极手段）](#方法八全网扫描--host-头碰撞终极手段)
- [自动化工具汇总](#自动化工具汇总)
- [验证真实 IP 的标准动作](#验证真实-ip-的标准动作)
- [防守方自查清单](#防守方自查清单)
- [工具选型总结](#工具选型总结)

---

## 核心思路

**绕过 CDN 找真实 IP 的核心思路**：优先查历史解析记录与子域名，其次利用证书透明度、邮件头泄露、SSRF 回显与全网测绘平台交叉验证，最后用 `curl -H "Host: xxx"` 完成确认。

主域名解析在静态阶段往往无解，必须从**旁路信息**切入。

---

## 方法总览（按命中率排序）

| 方法 | 原理 | 代表工具/平台 | 命中率 | 难度 |
|---|---|---|---|---|
| **历史 DNS 解析** | 域名接入 CDN 前的 A 记录被第三方存档 | SecurityTrails、ViewDNS、DNSDB | ★★★★★ | 低 |
| **子域名旁路** | mail/ftp/direct/origin 等子域名常裸奔 | subfinder、dig、OneForAll | ★★★★★ | 低 |
| **证书透明度日志** | 源站或早期证书记录含 IP 信息 | crt.sh、Censys | ★★★☆ | 低 |
| **网络空间测绘** | 用 favicon hash/证书/标题在全网反查 | FOFA、Shodan、Hunter、Quake | ★★★★ | 中 |
| **邮件头泄露** | 业务邮件从源站直接发出 | 注册账号 + 查看原始邮件头 | ★★★☆ | 低 |
| **源码与备份泄露** | .git/.env/备份文件暴露内网 IP | gau、dirsearch、GitHub 监控 | ★★☆ | 中 |
| **IPv6 直连** | AAAA 记录常被运维遗漏 | dig AAAA、curl -6 | ★★ | 低 |
| **全网扫描 + Host 头** | 对云厂商 IP 段逐个发 Host 验证 | masscan + httpx + 脚本 | ★★ | 高 |

---

## 方法一：历史 DNS 解析（首选）

网站几乎都是先有服务器、后上 CDN，接入前的 A 记录会被被动 DNS 平台永久存档。

**SecurityTrails 实操**：访问 `securitytrails.com`，输入目标域名，进入 **Historical DNS**，查看最早时间点的 A 记录——越早的记录越接近源站真实 IP。

**其他可选平台**：

- **ViewDNS.info** — "IP History" 功能免费直查
- **DNSDB (Farsight)** — 学术/商业级被动 DNS 数据库
- **Whoisxmlapi** — DNS 历史与关联域名综合查询

---

## 方法二：子域名旁路

CDN 通常只代理主域和 www，`mail`、`ftp`、`direct`、`origin`、`cpanel`、`dev`、`staging` 这类子域名经常直接解析到源站。**这是实际渗透测试中使用频率最高的方法。**

一条命令完成枚举 + 解析对比：

```bash
# 先收集子域名
subfinder -d example.com -all -silent > subs.txt

# 批量解析，与主域名 CDN IP 段不同的即为嫌疑源站
while read sub; do
  ip=$(dig +short "$sub" | grep -E '^[0-9]+\.' | tail -1)
  echo "$sub -> $ip"
done < subs.txt | sort -u -t'>' -k2
```

对老站或数据残缺的站点，用 dig 手工补查常见子域名：

```bash
for sub in mail ftp direct origin cpanel dev test staging admin api; do
  dig +short "$sub.example.com" | grep -v "^;" | xargs -I{} echo "$sub: {}"
done
```

---

## 方法三：证书透明度日志

源站与 CDN 之间建立 TLS 连接时，源站证书或早期申请的证书会被录入 CT 日志。在 **crt.sh** 输入域名后，证书详情的 "IP Address" 字段有时直接记载源站 IP；也可拿到证书序列号后去 Censys/Shodan 反查部署该证书的 IP 段。

```bash
# crt.sh 批量拉取证书关联的子域名与 IP
curl -s 'https://crt.sh/?q=%25.example.com&output=json' \
  | jq -r '.[].name_value' | sort -u
```

拿到证书序列号或 SAN 后，去 Censys 证书搜索中 `parsed.names: example.com` 反向定位。

---

## 方法四：网络空间测绘平台（FOFA/Shodan 语法直接可用）

测绘平台的核心价值是**用特征反查 IP**：favicon 哈希、证书 CN、页面标题、公司名备案。

**FOFA 常用语法**：

```
# 按域名反查所有关联资产
domain="example.com"
host=".example.com"

# 按组织证书反查
cert="Example Inc"

# favicon 哈希（测试环境常复用主站图标）
icon_hash="-123456789"
```

**Shodan 常用语法**：

```
ssl.cert.subject.cn:"example.com"
http.title:"Example Login"
http.favicon.hash:-123456789
org:"Example Corp"
```

**favicon 哈希计算**（Python，需 mmh3 库）：

```python
import mmh3, requests, codecs

response = requests.get('https://example.com/favicon.ico')
favicon = codecs.encode(response.content, 'base64')
hash_value = mmh3.hash(favicon)

print(f'http.favicon.hash:{hash_value}')   # 直接用于 Shodan
print(f'icon_hash="{hash_value}"')         # 直接用于 FOFA
```

---

## 方法五：邮件头溯源

网站触发邮件（注册验证、密码重置）时，若发件服务器未走 CDN 出口，邮件头的 `Received: from` 字段会暴露源站 IP 或同网段 IP。

**操作步骤**：注册测试账号 → 收到邮件后在邮箱客户端查看"原始邮件/显示源码" → 定位 `Received: from` 行 → 提取最内层 IP（非邮件服务商中继）→ 结合 IP 反查域名确认归属。

```bash
# 拿到候选 IP 后反查绑定过的域名（排除邮件服务商）
host 1.2.3.4
# 或用在线反查：ip138.com、viewdns.info/reverseip
```

---

## 方法六：源码泄露与配置文件

- **目录扫描**：`gau example.com | grep -E "\.(env|bak|sql|zip|tar|gz)"` 或用 dirsearch 扫 `.git`、`backup`、`database.sql`
- **GitHub/Gitee 监控**：搜索 `domain:example.com`、`@example.com password`、`example.com access_key` 等关键词，配置文件里常见内网 IP 与数据库地址
- **APP/小程序反编译**：jadx 反编译 APK，全局搜索 `http://`、`accessKey`，测试环境 API 常直接指向源站

---

## 方法七：SSRF 与错误页面

部分网站存在 SSRF 或配置不当，在错误页面、API 响应、回调接口中直接回显内网或源站 IP。

**探测点包括**：上传接口的回调 URL、Webhook 测试、PDF 导出、图片代理等。

---

## 方法八：全网扫描 + Host 头碰撞（终极手段）

前述方法都失败时，可对目标可能所在的云厂商/IDC IP 段做全网 HTTP 探测，用 Host 头匹配内容确认源站。

```bash
# 假设源站在 AWS，拉取其 IP 段
wget https://ip-ranges.amazonaws.com/ip-ranges.json -O ranges.json

# masscan 快速探测 HTTP 端口
masscan -p 80,443,8080,8443 -iL aws_ips.txt --rate=50000 --open -oG http_open.txt

# httpx 发送 Host 头验证
cat http_open.txt | awk '{print $4 ":" $3}' | \
  httpx -silent -host example.com -title -tech-detect
```

返回标题与技术栈与主站一致的 IP，即为源站。

---

## 自动化工具汇总

| 工具 | 用途 | 地址 |
|---|---|---|
| **CloudFlair** | 基于 Censys SSL 证书数据自动定位 Cloudflare 源站 | github.com/christophetd/cloudflair |
| **CloakQuest3r** | Python 工具，自动分析 Cloudflare 保护的源站 IP | github.com/spyboy-productions/CloakQuest3r |
| **bypass-firewalls-by-DNS-history** | 基于 DNS 历史记录自动绕过防火墙/CDN | github.com/vincentcox/bypass-firewalls-by-DNS-history |
| **FlareHunter** | 异步多线程检查多域名对应源站 IP | GitHub Topic: real-ip-cloudflare |

**CloudFlair 使用示例**：

```bash
python cloudflair.py myvulnerable.site
# 自动通过 Censys 查找匹配证书的 IPv4 主机，输出候选源站 IP 列表
```

---

## 验证真实 IP 的标准动作

拿到候选 IP 后，**必须用 Host 头验证**——CDN 节点 IP 用同样方法访问会返回默认页或错误页，而源站会返回正常内容：

```bash
# 核心验证命令
curl -I http://候选IP -H "Host: example.com"
# 若返回的状态码、Server 头、页面内容与直接访问域名一致，即确认为源站

# 辅助判断：源站通常开放管理端口，CDN 节点只开 80/443
nmap -p 22,80,443,3306,6379,8080 候选IP -Pn
```

**进一步确认技巧**：

- 对比 `curl http://IP/xxx`（不带 Host）与 `curl -H "Host: 域名" http://IP/xxx` 的响应差异
- 检查 IP 的 TLS 证书是否包含目标域名
- 用 IP 反查域名，看是否绑定多个同主体站点

---

## 防守方自查清单

- [ ] 接入 CDN 后**立即更换源站 IP**，旧 IP 一旦泄露永久存档
- [ ] 源站防火墙白名单只放行 CDN 回源 IP 段，其余流量直接丢弃
- [ ] 检查所有 DNS 记录类型，确保 A、AAAA、CNAME 无遗漏直连
- [ ] 源站证书改用自签名或内部 CA，避免被 CT 日志索引
- [ ] 邮件服务与源站分离，或通过独立的发信中继
- [ ] 子域名统一纳入 CDN 管理，杜绝"裸奔"旁路
- [ ] **配合 Authenticated Origin Pulls**：使用自定义 TLS 客户端证书做回源认证，而非仅依赖 IP 白名单

---

## 工具选型总结

| 场景 | 推荐组合 |
|---|---|
| 被动查询 | SecurityTrails + crt.sh |
| 主动枚举 | subfinder + dig |
| 全网反查 | FOFA / Shodan 测绘语法 |
| 自动化 | CloudFlair / CloakQuest3r |
| 验证确认 | `curl -H "Host: 域名" http://IP` |

---

## 参考资源

- [CloudFlair — 基于全网扫描数据的 Cloudflare 源站定位](https://github.com/christophetd/cloudflair)
- [bypass-firewalls-by-DNS-history — DNS 历史绕过脚本](https://github.com/vincentcox/bypass-firewalls-by-DNS-history)
- [Trickest — Cloudflare Origin IP Finder & WAF Bypass](https://trickest.com/library/cloudflare-origin-ip-finder-waf-bypass)
- [Relying solely on IP Allowlisting with Cloudflare is WRONG](https://javan.de/relying-solely-on-ip-allowlisting-with-cloudflare-is-wrong)
- [Cloudflare IP Ranges 官方文档](https://developers.cloudflare.com/fundamentals/concepts/cloudflare-ip-addresses)

---

> 📌 **持续更新**：本手册会随 CDN 技术与攻防实践持续更新，欢迎提 Issue / PR 补充新方法与实战案例。
>
> ⭐ 如果本手册对你有帮助，欢迎 Star 支持！

---

> 📄 注：本文部分内容由 AI 生成，无法确保真实准确，仅供参考。
