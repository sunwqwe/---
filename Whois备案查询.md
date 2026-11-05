# 🔎 Whois 查询与备案信息收集手册

> 域名归属与网站主体身份的权威查询方法合集，适用于**授权渗透测试、资产测绘与安全自查**。

⚠️ **免责声明**：本仓库内容仅用于合法授权的安全测试与防御自查。未经授权对目标进行信息收集可能违反《网络安全法》《数据安全法》及相关法律法规，使用者需自行承担一切法律责任。

---

## 📖 目录

- [一、Whois 查询](#一whois-查询)
  - [1.1 Whois 能查到什么](#11-whois-能查到什么)
  - [1.2 在线查询平台对比](#12-在线查询平台对比)
  - [1.3 命令行查询（whois / RDAP）](#13-命令行查询whois--rdap)
  - [1.4 Whois 历史记录查询](#14-whois-历史记录查询)
  - [1.5 隐私保护的影响](#15-隐私保护的影响)
- [二、备案查询](#二备案查询)
  - [2.1 工信部备案查询](#21-工信部备案查询)
  - [2.2 公安备案查询](#22-公安备案查询)
  - [2.3 第三方备案查询平台](#23-第三方备案查询平台)
  - [2.4 批量查询方案](#24-批量查询方案)
- [三、IP 归属地与反查](#三ip-归属地与反查)
- [四、信息交叉验证](#四信息交叉验证)
- [五、常见问题与排查](#五常见问题与排查)
- [六、防守视角：Whois 信息暴露面收敛](#六防守视角whois-信息暴露面收敛)
- [七、工具选型总结](#七工具选型总结)

---

## 一、Whois 查询

### 1.1 Whois 能查到什么

Whois 是查询域名注册信息的标准协议，可获取：

| 字段 | 说明 |
|---|---|
| **注册人** | 个人姓名或公司主体（开启隐私保护时显示 `REDACTED FOR PRIVACY`） |
| **注册商** | 域名从哪家公司注册 |
| **注册/到期时间** | 判断域名年龄与续费紧迫性 |
| **DNS 服务器** | 当前解析指向的 NS 主机 |
| **域名状态** | 正常、锁定、赎回期、待删除 |
| **联系邮箱/电话** | 注册人联系方式（未开隐私保护时） |

> 📌 自 2025 年 1 月 28 日起，ICANN 已正式淘汰传统 WHOIS 协议，新协议 **RDAP** 成为标准替代方案<span data-allow-html class="search-source-placeholder" data-keys="turn1search6,turn1search3" data-label="www.icann.org +1"><span class="search-source-item-label"><a data-allow-html href="https://www.icann.org/en/announcements/details/icann-update-launching-rdap-sunsetting-whois-27-01-2025-en" target="_blank" rel="noopener noreferrer">www.icann.org</a></span><span class="source-item-num-count"> +1</span></span>。

### 1.2 在线查询平台对比

| 平台 | 地址 | 特色 |
|---|---|---|
| **ICANN Lookup** | lookup.icann.org | **官方权威入口**，走 RDAP，适合单域名快速查询<span data-allow-html class="search-source-placeholder" data-keys="turn1search0" data-label="lookup.icann.org"><span class="search-source-item-label"><a data-allow-html href="https://lookup.icann.org" target="_blank" rel="noopener noreferrer">lookup.icann.org</a></span><span class="source-item-num-count"></span></span> |
| **站长之家 Whois** | whois.chinaz.com | 中文界面，支持 1500+ 后缀，附 RDAP 查询<span data-allow-html class="search-source-placeholder" data-keys="turn0search2" data-label="whois.chinaz.com"><span class="search-source-item-label"><a data-allow-html href="https://whois.chinaz.com/" target="_blank" rel="noopener noreferrer">whois.chinaz.com</a></span><span class="source-item-num-count"></span></span> |
| **爱站网 Whois** | whois.aizhan.com | 国内老牌，数据较全 |
| **Who.is** | who.is | 国际站，支持域名/IP/ASN |
| **whois.com** | whois.com | 国际通用 |
| **阿里云 Whois** | whois.aliyun.com | 中文，国内域名友好 |
| **r.box RDAP** | r.box | 国产 RDAP 查询工具<span data-allow-html class="search-source-placeholder" data-keys="turn2search0" data-label="r.box"><span class="search-source-item-label"><a data-allow-html href="https://r.box/" target="_blank" rel="noopener noreferrer">r.box</a></span><span class="source-item-num-count"></span></span> |

### 1.3 命令行查询（whois / RDAP）

**传统 whois 命令**（Linux / macOS 原生支持）：

```bash
# 标准查询
whois example.com

# 直接查注册局（.com/.net thin 模式）
whois -h whois.verisign-grs.com example.com

# 通过 IANA 根 WHOIS 查 referral
printf 'example.com\r\n' | nc whois.iana.org 43
```

**RDAP 命令**（推荐，JSON 结构化，适合脚本处理）：

```bash
# 通用入口，自动路由到对应注册局
curl -s https://rdap.org/domain/example.com | jq

# 直接查 .com 注册局
curl -s https://rdap.verisign.com/com/v1/domain/example.com | jq

# 查询 IP 段归属（RIR）
curl -s https://rdap.arin.net/registry/ip/8.8.8.8 | jq
```

> 💡 RDAP 优点：HTTPS + JSON 格式，字段结构化稳定，适合自动化巡检；传统 whois 是自由文本，各注册局格式不统一，写脚本解析维护成本高<span data-allow-html class="search-source-placeholder" data-keys="turn2search6" data-label="developer.aliyun.com"><span class="search-source-item-label"><a data-allow-html href="https://developer.aliyun.com/article/1771617" target="_blank" rel="noopener noreferrer">developer.aliyun.com</a></span><span class="source-item-num-count"></span></span>。

### 1.4 Whois 历史记录查询

历史 Whois 在域名交易、仲裁、溯源场景下非常关键。主流工具：

| 工具 | 特色 | 地址 |
|---|---|---|
| **Whoisology** | 收录 1999 年至今记录，支持邮箱/姓名反查 | whoisology.com |
| **DomainTools** | 老牌、数据最全，含 WHOIS History Panel 时间轴 | domaintools.com |
| **SecurityTrails** | DNS + WHOIS 历史综合查询 | securitytrails.com |
| **Wayback Machine** | 网页快照存档，免费 | web.archive.org |

**典型应用场景**：

- 判断域名是否被倒卖过（注册邮箱短期多次变更）
- 溯源钓鱼网站背后的真实注册人
- 域名仲裁取证（保存历史快照作为证据）

### 1.5 隐私保护的影响

开启隐私保护后，公开查询只能看到：

- ✅ 注册商、注册/到期时间、DNS 服务器、域名状态
- ❌ 注册人姓名、邮箱、电话、地址（显示 `REDACTED FOR PRIVACY`）

**合规联系真实持有人的途径**：

1. 通过注册商"联系域名持有人"表单转交邮件
2. 向注册商 abuse 邮箱提交滥用举报
3. 走 UDRP（统一域名争议解决政策）或司法程序
4. 域名已建站的，直接查看网站底部备案号、公司名<span data-allow-html class="search-source-placeholder" data-keys="turn2search2" data-label="www.gj0.com"><span class="search-source-item-label"><a data-allow-html href="https://www.gj0.com/thread-2440.html" target="_blank" rel="noopener noreferrer">www.gj0.com</a></span><span class="source-item-num-count"></span></span>

> ⚠️ 任何声称"付费即可反查隐藏 Whois 真实注册人"的服务，要么数据来自历史快照，要么来源不明，**不建议使用**<span data-allow-html class="search-source-placeholder" data-keys="turn2search2" data-label="www.gj0.com"><span class="search-source-item-label"><a data-allow-html href="https://www.gj0.com/thread-2440.html" target="_blank" rel="noopener noreferrer">www.gj0.com</a></span><span class="source-item-num-count"></span></span>。

---

## 二、备案查询

### 2.1 工信部备案查询

**官方权威入口**：[https://beian.miit.gov.cn](https://beian.miit.gov.cn)<span data-allow-html class="search-source-placeholder" data-keys="turn0search0" data-label="beian.miit.gov.cn"><span class="search-source-item-label"><a data-allow-html href="https://beian.miit.gov.cn?spm=d3d3lnlpd3vidxkuy29tl3byb2r1y3qvzgv0ywlslzk3oduwmty4os5odg1s.d3d3lnlpd3vidxkuy29tl3byb2r1y3qvzgv0ywlslzk1mjq5odu1mi5odg1s.d3d3lnlpd3vidxkuy29tl3nob3bkzxrhawwvnzu3mdk4xzeuahrtba==.d3d3lnlpd3vidxkuy29tl3nob3bkzxrhawwvnzu3mdk4xzquahrtbd9zb3j0pte=/" target="_blank" rel="noopener noreferrer">beian.miit.gov.cn</a></span><span class="source-item-num-count"></span></span>

**查询步骤**：

1. 打开 `beian.miit.gov.cn`，点击"ICP 备案查询"
2. 输入域名（**不带 `www` 和 `http://`**，如 `example.com`）
3. 完成滑块验证
4. 查看备案详情

**支持三种查询方式**<span data-allow-html class="search-source-placeholder" data-keys="turn0search7,turn0search13" data-label="www.beianyuantou.com +1"><span class="search-source-item-label"><a data-allow-html href="https://www.beianyuantou.com/beian-number-lookup" target="_blank" rel="noopener noreferrer">www.beianyuantou.com</a></span><span class="source-item-num-count"> +1</span></span>：

- 按域名查询（最常用，子域名共用主域名备案）
- 按备案号查询（输入完整备案号如"京ICP备XXXXXXXX号"）
- 按主办单位名称查询（查看企业名下所有备案域名）

**返回字段解读**<span data-allow-html class="search-source-placeholder" data-keys="turn0search13" data-label="www.kd.cn"><span class="search-source-item-label"><a data-allow-html href="https://www.kd.cn/ask/501313.html" target="_blank" rel="noopener noreferrer">www.kd.cn</a></span><span class="source-item-num-count"></span></span>：

| 字段 | 含义 |
|---|---|
| ICP 备案号 | 格式"京ICP备XXXXXX号-X" |
| 主办单位名称 | 备案主体全称（企业/事业单位/个人） |
| 主办单位性质 | 企业/个人/政府/事业单位/社会团体 |
| 网站名称 | 备案填报的网站名 |
| 审核通过时间 | 备案审核通过日期 |
| 网站域名 | 备案关联的域名列表 |

### 2.2 公安备案查询

完成 ICP 备案后，多数网站还需办理**公安联网备案**（两套独立系统）：

**官方入口**：[https://www.beian.gov.cn](https://www.beian.gov.cn)<span data-allow-html class="search-source-placeholder" data-keys="turn1search2" data-label="www.beian.gov.cn"><span class="search-source-item-label"><a data-allow-html href="https://www.beian.gov.cn/" target="_blank" rel="noopener noreferrer">www.beian.gov.cn</a></span><span class="source-item-num-count"></span></span>

**查询步骤**<span data-allow-html class="search-source-placeholder" data-keys="turn1search8" data-label="www.feidong.gov.cn"><span class="search-source-item-label"><a data-allow-html href="https://www.feidong.gov.cn/ahzwfwbsdt/tpl/4351?ssqdId=4028e69c942c00d70194920e0ed834aa&amp;ssqdCode=466d9784c69443259ebad04de618d91b" target="_blank" rel="noopener noreferrer">www.feidong.gov.cn</a></span><span class="source-item-num-count"></span></span>：

1. 访问"全国互联网安全管理服务平台"（beian.gov.cn）公共查询模块
2. 选择"网站名称"或"域名"选项
3. 输入关键词 + 验证码即可查询

> 💡 合规网站底部通常同时展示 **ICP 备案号 + 公安备案号**，并分别链接到对应系统，核验网站真实性时建议**两个号都查一遍**<span data-allow-html class="search-source-placeholder" data-keys="turn0search7" data-label="www.beianyuantou.com"><span class="search-source-item-label"><a data-allow-html href="https://www.beianyuantou.com/beian-number-lookup" target="_blank" rel="noopener noreferrer">www.beianyuantou.com</a></span><span class="source-item-num-count"></span></span>。

### 2.3 第三方备案查询平台

| 平台 | 地址 | 特色 |
|---|---|---|
| **站长之家 ICP 备案查询** | icp.chinaz.com | 数据对接工信部，支持域名/公司名查询 |
| **站长之家 APP 备案查询** | icp.chinaz.com/app | 查询 APP 备案信息（2023 年新规）<span data-allow-html class="search-source-placeholder" data-keys="turn0search14" data-label="www.chinaz.com"><span class="search-source-item-label"><a data-allow-html href="https://www.chinaz.com/2023/0908/1557679.shtml" target="_blank" rel="noopener noreferrer">www.chinaz.com</a></span><span class="source-item-num-count"></span></span> |
| **站长之家小程序备案查询** | icp.chinaz.com/miniapp | 小程序备案查询<span data-allow-html class="search-source-placeholder" data-keys="turn0search14" data-label="www.chinaz.com"><span class="search-source-item-label"><a data-allow-html href="https://www.chinaz.com/2023/0908/1557679.shtml" target="_blank" rel="noopener noreferrer">www.chinaz.com</a></span><span class="source-item-num-count"></span></span> |
| **爱站网备案查询** | beian.aizhan.com | 支持域名/公司名/备案号 |
| **CHAICP 全国备案查询** | chaicp.com | 老牌备案查询站点<span data-allow-html class="search-source-placeholder" data-keys="turn0search3" data-label="blog.51cto.com"><span class="search-source-item-label"><a data-allow-html href="https://blog.51cto.com/u_17615043/14426368" target="_blank" rel="noopener noreferrer">blog.51cto.com</a></span><span class="source-item-num-count"></span></span> |
| **腾讯云/阿里云备案查询** | 云控制台 | 查询接入商备案状态 |

> ⚠️ 第三方工具的数据来自对工信部系统的对接，存在 1-3 天延迟。涉及交易、租用备案域名等场景，**必须以工信部官网当次查询结果为准**<span data-allow-html class="search-source-placeholder" data-keys="turn0search10,turn0search7" data-label="idctop.com +1"><span class="search-source-item-label"><a data-allow-html href="https://idctop.com/article/570887.html" target="_blank" rel="noopener noreferrer">idctop.com</a></span><span class="source-item-num-count"> +1</span></span>。

### 2.4 批量查询方案

**站长之家实时批量查询平台**（付费）<span data-allow-html class="search-source-placeholder" data-keys="turn0search5" data-label="m.chinaz.com"><span class="search-source-item-label"><a data-allow-html href="https://m.chinaz.com/2022/1014/1456794.shtml" target="_blank" rel="noopener noreferrer">m.chinaz.com</a></span><span class="source-item-num-count"></span></span>：

- 地址：`tool.chinaz.com/batchquery`
- 单次支持 5 万条数据
- 支持自定义导出字段（备案号、单位性质、审核日期等）
- 支持 Whois / 备案 / IP / HTTP 状态一键批量

**备案大数据工具**（获取最新备案动态）<span data-allow-html class="search-source-placeholder" data-keys="turn0search8" data-label="m.chinaz.com"><span class="search-source-item-label"><a data-allow-html href="https://m.chinaz.com/2021/0811/1289827.shtml" target="_blank" rel="noopener noreferrer">m.chinaz.com</a></span><span class="source-item-num-count"></span></span>：

- 地址：`yx.chinaz.com/icp/latestRegisteredDomain`
- 可按今日/最近 7 天/行业/省份筛选最新备案域名
- 支持同邮箱、同电话关联域名挖掘

---

## 三、IP 归属地与反查

Whois 和备案之外，IP 层面的查询是第三把钥匙。

### 3.1 获取目标 IP

```bash
# 命令行获取解析 IP
dig +short example.com
nslookup example.com

# 查询本机公网 IP
curl myip.ipip.net        # 文本格式
curl ipinfo.io            # JSON 格式
curl ip.sb                # 极简输出
curl ifconfig.me
```

### 3.2 IP 归属地查询平台

| 平台 | 地址 | 特色 |
|---|---|---|
| **IP138** | ip138.com | 老牌站点，国内 IP 数据库丰富<span data-allow-html class="search-source-placeholder" data-keys="turn1search1" data-label="www.ip138.com"><span class="search-source-item-label"><a data-allow-html href="https://www.ip138.com/" target="_blank" rel="noopener noreferrer">www.ip138.com</a></span><span class="source-item-num-count"></span></span> |
| **IPIP.net** | ipip.net | 国内 IP 数据库专业服务商，精度高，可定位到区县<span data-allow-html class="search-source-placeholder" data-keys="turn1search7" data-label="www.10000vps.com"><span class="search-source-item-label"><a data-allow-html href="http://www.10000vps.com/article/661.html" target="_blank" rel="noopener noreferrer">www.10000vps.com</a></span><span class="source-item-num-count"></span></span> |
| **IPInfo.io** | ipinfo.io | 国际主流，含 ASN 与运营商信息<span data-allow-html class="search-source-placeholder" data-keys="turn1search4" data-label="blog.alanwei.com"><span class="search-source-item-label"><a data-allow-html href="https://blog.alanwei.com/blog/2026/09/20/ip-geo/" target="_blank" rel="noopener noreferrer">blog.alanwei.com</a></span><span class="source-item-num-count"></span></span> |
| **ip.sb** | ip.sb | 极简界面，无广告<span data-allow-html class="search-source-placeholder" data-keys="turn1search4" data-label="blog.alanwei.com"><span class="search-source-item-label"><a data-allow-html href="https://blog.alanwei.com/blog/2026/09/20/ip-geo/" target="_blank" rel="noopener noreferrer">blog.alanwei.com</a></span><span class="source-item-num-count"></span></span> |
| **ip-api.com** | ip-api.com | 免费开放 API，返回 JSON（45 次/分钟限流）<span data-allow-html class="search-source-placeholder" data-keys="turn1search7" data-label="www.10000vps.com"><span class="search-source-item-label"><a data-allow-html href="http://www.10000vps.com/article/661.html" target="_blank" rel="noopener noreferrer">www.10000vps.com</a></span><span class="source-item-num-count"></span></span> |
| **ipinfo 免费版** | ipinfo.io | 每日 1000 次免费 API<span data-allow-html class="search-source-placeholder" data-keys="turn1search7" data-label="www.10000vps.com"><span class="search-source-item-label"><a data-allow-html href="http://www.10000vps.com/article/661.html" target="_blank" rel="noopener noreferrer">www.10000vps.com</a></span><span class="source-item-num-count"></span></span> |
| **百度 IP 查询** | qifu.baidu.com | 国内访问稳定<span data-allow-html class="search-source-placeholder" data-keys="turn1search10" data-label="www.ebingou.cn"><span class="search-source-item-label"><a data-allow-html href="https://www.ebingou.cn/gongju/22162.html" target="_blank" rel="noopener noreferrer">www.ebingou.cn</a></span><span class="source-item-num-count"></span></span> |

### 3.3 IP 反查域名

输入 IP，返回该 IP 上绑定的所有域名列表：

- **ViewDNS.info** — `viewdns.info/reverseip/`
- **站长之家 IP 反查** — `tool.chinaz.com/same/`
- **IP138** — ip138.com 提供 IP 反查功能
- **微步在线** — x.threatbook.com（威胁情报综合）

**使用场景**：排查恶意 IP、分析同主体站点、CDN 绕过后确认源站<span data-allow-html class="search-source-placeholder" data-keys="turn1search13" data-label="idctop.com"><span class="search-source-item-label"><a data-allow-html href="https://idctop.com/article/574277.html" target="_blank" rel="noopener noreferrer">idctop.com</a></span><span class="source-item-num-count"></span></span>。

---

## 四、信息交叉验证

### 4.1 三把钥匙联动

判断一个网站的真实性，建议**三线交叉验证**<span data-allow-html class="search-source-placeholder" data-keys="turn1search13" data-label="idctop.com"><span class="search-source-item-label"><a data-allow-html href="https://idctop.com/article/574277.html" target="_blank" rel="noopener noreferrer">idctop.com</a></span><span class="source-item-num-count"></span></span>：

```
Whois（域名注册信息） → ICP 备案（网站主体身份） → IP 归属地（服务器位置）
```

**典型排查示例**（仿冒网站识别）：

| 排查项 | 正规站特征 | 可疑信号 |
|---|---|---|
| Whois 域名年龄 | 注册多年 | 注册时间短（3 个月内） |
| ICP 备案主体 | 与页面公司名一致 | 备案信息不存在，或主体与页面不符 |
| IP 归属地 | 与宣称业务地区匹配 | 国内公司但服务器在境外 IDC |
| 公安备案 | 已办理 | 仅有 ICP 备案，无公安备案 |

### 4.2 一键查询工具

- **站长之家综合查询**：`tool.chinaz.com` — 输入域名同时返回 IP、备案、Whois、HTTP 状态
- **微步在线**：`x.threatbook.com` — 威胁情报综合，含 Whois 历史、IP 反查、样本分析
- **奇安信 TI**：`ti.qianxin.com` — 国内威胁情报平台

---

## 五、常见问题与排查

**Q1：Whois 显示 REDACTED FOR PRIVACY 怎么办？**
通过注册商联系表单、abuse 邮箱、UDRP 或司法程序等合规途径联系持有人，不建议使用"付费反查"服务<span data-allow-html class="search-source-placeholder" data-keys="turn2search2" data-label="www.gj0.com"><span class="search-source-item-label"><a data-allow-html href="https://www.gj0.com/thread-2440.html" target="_blank" rel="noopener noreferrer">www.gj0.com</a></span><span class="source-item-num-count"></span></span>。

**Q2：备案查询显示"暂无数据"是怎么回事？**
可能原因<span data-allow-html class="search-source-placeholder" data-keys="turn0search13" data-label="www.kd.cn"><span class="search-source-item-label"><a data-allow-html href="https://www.kd.cn/ask/501313.html" target="_blank" rel="noopener noreferrer">www.kd.cn</a></span><span class="source-item-num-count"></span></span>：

- 域名未备案（使用境外服务器或违规运营）
- 备案信息还在审核中（未公开）
- 输入了带 `www` 或 `http://` 的域名（应输入主域名）
- 备案已被注销

**Q3：第三方工具显示已备案，能直接信吗？**
不能。第三方数据存在 1-3 天延迟，涉及金钱往来的场景**必须以工信部官网当次查询为准**<span data-allow-html class="search-source-placeholder" data-keys="turn0search7,turn0search10" data-label="www.beianyuantou.com +1"><span class="search-source-item-label"><a data-allow-html href="https://www.beianyuantou.com/beian-number-lookup" target="_blank" rel="noopener noreferrer">www.beianyuantou.com</a></span><span class="source-item-num-count"> +1</span></span>。

**Q4：查询时输入域名要带 www 吗？**
不需要。工信部备案系统记录的是主域名，输入 `example.com` 即可<span data-allow-html class="search-source-placeholder" data-keys="turn0search10" data-label="idctop.com"><span class="search-source-item-label"><a data-allow-html href="https://idctop.com/article/570887.html" target="_blank" rel="noopener noreferrer">idctop.com</a></span><span class="source-item-num-count"></span></span>。

**Q5：备案号里的地区简称代表什么？**
代表备案主体所在省级行政区（如"京"=北京、"沪"=上海），不限制网站访问区域<span data-allow-html class="search-source-placeholder" data-keys="turn0search7" data-label="www.beianyuantou.com"><span class="search-source-item-label"><a data-allow-html href="https://www.beianyuantou.com/beian-number-lookup" target="_blank" rel="noopener noreferrer">www.beianyuantou.com</a></span><span class="source-item-num-count"></span></span>。

**Q6：查到备案号主体和域名跟我的信息不符？**
可能是备案被冒用，可联系工信部进行核实，必要时向属地通信管理局举报<span data-allow-html class="search-source-placeholder" data-keys="turn0search7" data-label="www.beianyuantou.com"><span class="search-source-item-label"><a data-allow-html href="https://www.beianyuantou.com/beian-number-lookup" target="_blank" rel="noopener noreferrer">www.beianyuantou.com</a></span><span class="source-item-num-count"></span></span>。

---

## 六、防守视角：Whois 信息暴露面收敛

企业域名自查建议：

- [ ] **开启 Whois 隐私保护**（多数注册商免费提供）
- [ ] **注册商账户启用 2FA**，防止注册商账户被盗导致域名转移
- [ ] **使用企业专用邮箱**做域名注册联系人，避免个人邮箱暴露
- [ ] **定期在 Whoisology / DomainTools 查询自有域名历史**，发现异常变更
- [ ] **备案信息一致性自查**：确保 ICP 备案、公安备案、APP 备案主体信息一致
- [ ] **监控域名到期时间**，避免因续费疏忽被抢注
- [ ] **及时发现"同主体下未授权域名"**：用备案大数据工具反查同邮箱/电话备案的域名

---

## 七、工具选型总结

| 场景 | 推荐组合 |
|---|---|
| 单域名 Whois 快查 | ICANN Lookup 或 站长之家 Whois |
| 批量 Whois / RDAP 巡检 | `curl rdap.org/domain/xxx` + jq 脚本 |
| 历史注册信息溯源 | Whoisology + DomainTools |
| 国内网站主体核实 | 工信部备案系统 + 公安备案平台 |
| 批量备案查询 | 站长之家批量查询平台 |
| IP 归属地查询 | IPIP.net（国内）+ IPInfo.io（国际） |
| IP 反查域名 | ViewDNS.info + 站长之家 |

---

## 参考资源

- [ICANN Lookup 官方入口](https://lookup.icann.org)
- [ICANN RDAP 协议说明](https://www.icann.org/en/contracted-parties/registry-operators/registration-data-access-protocol)
- [工信部 ICP/IP 地址/域名信息备案管理系统](https://beian.miit.gov.cn)
- [全国互联网安全管理服务平台（公安备案）](https://www.beian.gov.cn)
- [站长之家工具集合](https://tool.chinaz.com)
- [RDAP Bootstrap 文件（IANA）](https://data.iana.org/rdap/dns.json)

---

> 📌 **持续更新**：本手册会随协议演进与查询平台变化持续更新，欢迎提 Issue / PR 补充新方法与实战案例。
>
> ⭐ 如果本手册对你有帮助，欢迎 Star 支持！

