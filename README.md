# 🔍 信息收集安全手册

> 适用于**授权渗透测试、红蓝攻防、SRC 挖洞与安全自查**的实用信息收集技术手册合集。

⚠️ **免责声明**：本仓库内容仅用于合法授权的安全测试与防御自查。未经授权对目标进行信息收集与测试可能违反《网络安全法》《数据安全法》及相关法律法规，使用者需自行承担一切法律责任。

---

## 📚 文档目录

| 文档 | 说明 |
|---|---|
| [敏感信息收集-Google-Baidu.md](./敏感信息收集-Google-Baidu.md) | Google / Baidu 搜索引擎敏感信息收集语法与实战案例（被动、零触碰） |
| [CDN绕过技术.md](./CDN绕过技术.md) | 绕过 CDN 找真实 IP 的 8 种方法、自动化工具与验证技巧 |
| [Whois备案查询.md](./Whois备案查询.md) | Whois 查询、ICP/公安备案查询、IP 归属地与反查方法合集 |
| [子域名收集.md](./子域名收集.md) | 子域名枚举的 8 大方法论、全工具链与完整工作流脚本 |
| [常用端口信息收集.md](./常用端口信息收集.md) | 端口扫描工具对比、高危端口利用路径、NSE 脚本与完整工作流 |
| [指纹识别.md](./指纹识别.md) | Web/CMS/中间件/WAF 指纹识别方法论、工具链与自定义指纹编写 |
| [敏感目录文件收集.md](./敏感目录文件收集.md) | 目录与文件爆破工具链、字典选型、403 绕过与完整工作流（主动、需授权） |

---

## 📖 内容概览

### 一、搜索引擎敏感信息收集（被动，零触碰）

- **Google 高级搜索语法**：`site:`、`inurl:`、`intitle:`、`filetype:` 等操作符详解
- **Baidu 高级搜索语法**：含中文特有书名号《》精确匹配技巧
- **实战案例**：敏感文件泄露 / 后台入口 / 开放目录 / GitHub 代码泄露 / 企业内部信息 / 网盘泄露
- **自动化工具**：GHDB、GShark、Pagodo 等工具使用

### 二、CDN 绕过技术（找真实 IP）

- **8 种绕过方法**：历史 DNS / 子域名旁路 / 证书透明度 / 空间测绘 / 邮件头溯源 / 源码泄露 / SSRF 回显 / 全网扫描
- **自动化工具**：CloudFlair、CloakQuest3r、bypass-firewalls-by-DNS-history
- **验证技巧**：Host 头碰撞验证、TLS 证书比对、端口特征判断
- **防守清单**：源站 IP 更换、回源白名单、CT 日志规避等自查项

### 三、Whois 查询与备案信息收集

- **Whois 查询**：在线平台对比、whois/RDAP 命令、历史记录溯源、隐私保护影响
- **备案查询**：工信部 ICP 备案、公安联网备案、APP/小程序备案、批量查询方案
- **IP 归属地与反查**：IPIP.net、IPInfo.io、ViewDNS 反查域名
- **交叉验证**：Whois + 备案 + IP 三线联动识别仿冒网站

### 四、子域名收集

- **8 大方法论**：被动聚合 / CT 日志 / DNS 爆破 / 排列组合 / 爬虫挖掘 / 搜索引擎 / AXFR / Takeover
- **全工具链**：subfinder、amass、OneForAll、dnsx、puredns、dnsgen、gotator
- **API Key 配置**：Censys / Shodan / SecurityTrails / VirusTotal / Chaos / BinaryEdge
- **存活验证**：dnsx 解析验证 → httpx HTTP 探活 → 泛解析过滤
- **完整脚本**：一站式 recon.sh 工作流，自动串联 5 个阶段

### 五、常用端口信息收集与利用

- **工具对比**：Nmap / masscan / RustScan / fscan / Zmap / Goby 六大工具横向评测
- **Nmap 速查**：主机发现、SYN/UDP 全端口、服务识别、NSE 漏洞脚本、防火墙规避
- **组合工作流**：masscan 全端口快扫 + nmap 服务识别的标准链路
- **端口分类**：文件共享 / 远程连接 / Web 中间件 / 数据库 / 邮件 / 网络协议 / 特殊服务
- **高危端口详解**：21 / 22 / 445 / 3306 / 6379 / 3389 / 8080 / 7001 / 27017 / 2375 / 9200 利用路径与 CVE
- **完整脚本**：portscan.sh 一站式工作流，自动串联 6 个阶段

### 六、指纹识别

- **指纹特征来源**：响应头 / Body / 静态路径 / Favicon / Cookie / TLS 证书九大特征位置
- **工具横向对比**：httpx / EHole / whatweb / Wappalyzer / fingers / TideFinger / kscan / goby
- **命令行详解**：httpx 批量打标、EHole 国内供应链、whatweb 单目标深挖
- **WAF 识别**：wafw00f 用法、常见 WAF 指纹特征表
- **测绘反查**：FOFA / Hunter / Shodan 语法、favicon mmh3 哈希计算
- **自定义指纹**：EHole JSON 格式、nuclei YAML 模板、编写原则
- **完整脚本**：fingerprint.sh 工作流，串联 httpx / EHole / wafw00f / whatweb

### 七、敏感目录与文件收集（主动，需授权）

- **与搜索引擎 Dork 的区别**：流量方向、隐蔽性、时效性、合规风险四维对比
- **工具横向对比**：dirsearch / ffuf / feroxbuster / gobuster / dirb / DirBuster / 御剑 / wfuzz
- **字典选型**：SecLists 按规模（common → medium → big）与按目的（API/备份/技术栈）双维选择
- **高价值路径**：版本控制 / 配置调试 / 后台管理 / 日志运行时 / 上传接口五类清单
- **假阳性与 403 绕过**：ffuf `-ac` 校准、SPA 过滤、X-Forwarded-For / 路径变体 / Host 头 / HTTP 方法绕过
- **API 扫描**：Swagger/OpenAPI 探测、Spring Boot Actuator、GraphQL 端点
- **联动挖掘**：gau 历史 URL、Arjun 参数发现、ParamSpider、git-dumper 源码提取
- **完整脚本**：dirscan.sh 一站式工作流，自动串联 6 个阶段

---

## 🚀 快速开始

1. 点击上方目录中的文档链接阅读完整内容
2. 将文档中的 `example.com` / `192.168.1.105` 替换为目标域名或 IP
3. 实战建议按以下完整链路组合使用：

```
① Whois + 备案查询（确认主体身份与资产边界）
        ↓
② 子域名收集（subfinder + crt.sh + amass + dnsx）
        ↓
③ CDN 绕过（定位关键资产的真实源站 IP）
        ↓
④ 端口扫描（masscan 全端口 + nmap 服务识别 + NSE 漏洞脚本）
        ↓
⑤ 指纹识别（httpx + EHole + whatweb + wafw00f）
        ↓
⑥ 敏感目录扫描（dirsearch + ffuf + feroxbuster，按指纹选字典）
        ↓
⑦ 搜索引擎 Dork（挖公开敏感文件、GitHub 泄露，可与 ⑥ 并行）
        ↓
⑧ 漏洞验证（按发现的路径与指纹选择 PoC）
```

> 💡 **节奏建议**：①②③ 为被动收集（零触碰），④⑤⑥⑦ 为主动收集（会产生流量），⑧ 为漏洞验证。主动阶段**务必在授权范围内执行**，并优先做风险最低的 ⑦（搜索引擎 Dork）。

## 🛡️ 防守方视角

如果你是防守方，建议重点关注以下自查章节：

- 敏感信息手册的 **[防守视角章节](./敏感信息收集-Google-Baidu.md#七防守视角如何防止被-dork)** —— 防止自有资产被搜索引擎 Dork
- CDN 绕过手册的 **[防守方自查清单](./CDN绕过技术.md#防守方自查清单)** —— 防止源站 IP 泄露
- Whois 手册的 **[Whois 信息暴露面收敛](./Whois备案查询.md#六防守视角whois-信息暴露面收敛)** —— 域名注册信息与备案一致性自查
- 子域名手册的 **[收敛子域名暴露面](./子域名收集.md#十一防守视角收敛子域名暴露面)** —— AXFR 加固、CT 监控、Takeover 风险治理
- 端口手册的 **[端口暴露面收敛](./常用端口信息收集.md#十防守视角端口暴露面收敛)** —— 最小暴露原则、高危服务默认配置加固
- 指纹手册的 **[指纹暴露面收敛](./指纹识别.md#十二防守视角指纹暴露面收敛)** —— 响应头抹除、调试端点关闭、favicon 策略
- 目录手册的 **[敏感路径暴露面收敛](./敏感目录文件收集.md#十一防守视角敏感路径暴露面收敛)** —— 禁止敏感文件上 Web、关闭目录列表、清理 .git、关闭 Swagger、WAF 爆破告警

---

> 📌 **持续更新**：本仓库会随攻防实践持续更新，欢迎提 Issue / PR 补充新方法与实战案例。
>
> ⭐ 如果本仓库对你有帮助，欢迎 Star 支持！

---

> 📄 注：本仓库部分内容由 AI 生成，无法确保真实准确，仅供参考。
