# 🔍 搜索引擎敏感信息收集手册（Google / Baidu 篇）

> 基于 Google Hacking 与百度高级搜索语法的敏感信息收集速查手册，适用于授权渗透测试、红蓝攻防与安全自查。
>
> ⚠️ **免责声明**：本手册仅用于合法授权的安全测试与防御自查。未经授权对目标进行信息收集与测试可能违反《网络安全法》《数据安全法》及相关法律法规，使用者需自行承担一切法律责任。

---

## 📖 目录

- [一、概述](#一概述)
- [二、Google 高级搜索语法](#二google-高级搜索语法)
- [三、Baidu 高级搜索语法](#三baidu-高级搜索语法)
- [四、敏感信息收集实战案例](#四敏感信息收集实战案例)
  - [4.1 敏感文件与文档泄露](#41-敏感文件与文档泄露)
  - [4.2 后台与登录入口](#42-后台与登录入口)
  - [4.3 开放目录与错误页面](#43-开放目录与错误页面)
  - [4.4 GitHub 代码泄露](#44-github-代码泄露)
  - [4.5 企业内部信息收集](#45-企业内部信息收集)
- [五、进阶技巧](#五进阶技巧)
- [六、自动化与工具联动](#六自动化与工具联动)
- [七、防守视角：如何防止被 Dork](#七防守视角如何防止被-dork)
- [八、参考资源](#八参考资源)

---

## 一、概述

搜索引擎索引了海量互联网页面与文件，其中相当一部分是企业和开发者**无意中暴露**的敏感信息：后台登录页、数据库备份、配置文件、网络拓扑图、内部通讯录等。

利用搜索引擎的**高级搜索语法（Search Operators）**精准检索这些暴露面，称为 **Google Hacking / Google Dorking**。其核心优势是：

- **被动收集**：所有查询均发向搜索引擎而非目标本身，不在目标日志中留痕；
- **零成本**：无需工具与环境，一条浏览器即可开始；
- **历史数据**：可发现已被删除但仍被索引的旧页面（配合缓存快照）。

据统计，超过半数的公开 GitHub 仓库存在不同程度的信息泄露（配置文件、API 密钥、硬编码口令等），搜索引擎是发现这些暴露面的第一入口。

---

## 二、Google 高级搜索语法

| 操作符 | 功能 | 示例 |
|---|---|---|
| `site:` | 限定搜索指定域名 | `site:example.com` |
| `inurl:` | URL 中包含关键字 | `inurl:admin` |
| `intitle:` | 页面标题包含关键字 | `intitle:"index of"` |
| `intext:` / `allintext:` | 正文包含关键字 | `intext:"password"` |
| `filetype:` / `ext:` | 限定文件后缀 | `filetype:pdf` |
| `cache:` | 查看 Google 缓存快照 | `cache:example.com` |
| `""` | 精确匹配，不拆词 | `"internal use only"` |
| `-` | 排除关键词（前面留空格） | `admin -login` |
| `*` | 通配符 | `site:*.example.com` |
| `|` / `OR` | 逻辑或 | `ftp \| sftp` |
| `..` | 数字范围（年份等） | `报告 2022..2024` |

**组合示例**：

```
site:example.com filetype:xls intext:"password"
```
→ 在目标域名下查找正文含 password 的 Excel 文件。

---

## 三、Baidu 高级搜索语法

百度支持大部分通用语法，并有中文特有功能：

| 操作符 | 功能 | 示例 |
|---|---|---|
| `site:` | 限定站点（不带 `http://`，不留空格） | `通讯录 site:example.com` |
| `inurl:` | URL 包含关键字 | `登录 inurl:admin` |
| `intitle:` | 标题包含关键字 | `intitle:后台登录` |
| `filetype:` | 限定文件格式 | `filetype:doc 网络拓扑` |
| `""` | 精确匹配不拆词 | `"员工信息表"` |
| `-` | 排除关键词（前面留空格） | `OA系统 -演示` |
| `《》` | **百度独有**：书名号精确匹配且不拆分 | `《XXX系统操作手册》` |

**组合示例**：

```
site:example.com filetype:xlsx 员工 通讯录
```
→ 查找目标站点下可能泄露的员工通讯录 Excel 表格。

---

## 四、敏感信息收集实战案例

> 💡 使用技巧：将 `example.com` 替换为目标主域，配合企业简称、内网域名（如 `corp.`、`oa.`、`mail.`）扩展搜索。

### 4.1 敏感文件与文档泄露

**通用敏感文件类型**：`xls / xlsx / csv / doc / pdf / sql / log / bak / conf / mdb / txt`

```
# 目标站点的 Excel 通讯录
site:example.com filetype:xls 通讯录
site:example.com filetype:xlsx intext:"姓名" intext:"手机"

# 数据库备份文件
site:example.com filetype:sql
site:example.com inurl:backup filetype:sql

# 配置文件泄露
site:example.com filetype:conf
site:example.com inurl:config
site:example.com filetype:log

# 网络拓扑 / 内部文档（不限站点，扩大范围）
"example.com" filetype:pdf 拓扑图
"某公司名称" filetype:doc 网络架构
"某公司名称" filetype:xls 员工
"某公司名称" filetype:pdf 招标
```

### 4.2 后台与登录入口

```
# 常见后台路径
site:example.com inurl:admin
site:example.com inurl:login
site:example.com inurl:manage
site:example.com intitle:登录
site:example.com intitle:管理后台

# 排除主站干扰，聚焦深链
site:example.com inurl:admin -inurl:www
site:example.com (login | admin | manage)
```

### 4.3 开放目录与错误页面

```
# 开放目录列表（Directory Listing）
intitle:"index of" site:example.com
intitle:"index of" "parent directory"
intitle:"index of" backup site:example.com

# phpinfo 调试页面
site:example.com inurl:phpinfo

# 错误页面泄露路径信息
site:example.com intext:"Warning" intext:"mysql"
site:example.com inurl:error
```

### 4.4 GitHub 代码泄露

开发者常将含数据库口令、SMTP 配置、API 密钥的代码误传到 GitHub，可通过 Google 或 GitHub 站内搜索定位：

```
# SMTP 邮箱配置泄露
site:github.com smtp @example.com
site:github.com smtp password
site:github.com "smtp.example.com" password

# 数据库连接串泄露
site:github.com "jdbc:mysql" example.com
site:github.com "example.com" password
site:github.com root password

# SVN / FTP 信息
site:github.com svn username password
site:github.com ftp password

# 云厂商密钥
site:github.com "example.com" access_key
site:github.com "example.com" secret_key
```

**GitHub 站内高级搜索语法**（在 github.com/search 使用）：

| 语法 | 功能 |
|---|---|
| `in:name 关键词` | 按仓库名搜索 |
| `in:readme 关键词` | 按 README 内容搜索 |
| `in:file 关键词` | 按文件内容搜索 |
| `user:用户名` | 限定用户/组织 |
| `language:java` | 限定开发语言 |
| `extension:properties` | 限定文件后缀（如配置文件） |

**示例**：
```
"example.com" password in:file language:java
smtp example.com in:file extension:properties
```

### 4.5 企业内部信息收集

```
# 邮箱与人员信息
"@example.com" 通讯录
"@example.com" filetype:xls

# 内部系统与测试环境
site:example.com inurl:test
site:example.com inurl:dev
"oa.example.com" OR "vpn.example.com" OR "mail.example.com"

# 对公网盘泄露（配合网盘搜索引擎）
"某公司名称" 网盘 密码
"某公司名称" 文件夹 分享
```

**网盘搜索平台**（输入公司名/关键词检索公开分享内容）：

| 平台 | 地址 |
|---|---|
| 凌风云搜索 | lingfengyun.com |
| 小白盘 | xiaobaipan.com |
| 大力盘 | dalipan.com |
| 大圣盘 | dashengpan.com |
| 云盘精灵 | yunpanjingling.com |

---

## 五、进阶技巧

### 5.1 多操作符组合的逻辑链

```
site:example.com filetype:xls intext:"password" -inurl:public
```
→ 目标域 + Excel 文件 + 正文含 password + 排除 public 路径

### 5.2 利用通配符发现子域

```
site:*.example.com -www
```
→ 列出被索引的所有子域名（排除 www 干扰）

### 5.3 Google 缓存找回已删除页面

```
cache:example.com/deleted_page.html
```
→ 页面虽已在目标站删除，但 Google 快照可能仍保留旧版本（含敏感信息）。

### 5.4 空间测绘引擎联动（延伸）

搜索引擎之外，FOFA / Hunter / Quake 等测绘平台可直接检索资产指纹：

```
# FOFA 语法示例
domain="example.com" && title="登录"
domain="example.com" && port="8080"
cert="Example Inc"
```

### 5.5 时间范围限定

- Google 搜索工具中可限定时间范围，发现近期新暴露内容；
- `..` 语法限定数字区间，如 `"报告" 2023..2024 filetype:pdf`。

---

## 六、自动化与工具联动

| 工具 | 用途 | 地址 |
|---|---|---|
| **GHDB** | Google Hacking 官方数据库，数千条分类 Dork | exploit-db.com/google-hacking-database |
| **GShark** | GitHub 敏感信息自动化监控，支持关键词订阅 | github.com/madneal/gshark |
| **gsd** | 命令行批量执行 Google Dork | github.com/TheSpeedX/gsd |
| **Pagodo** | Python 自动化 GHDB 批量扫描 | github.com/opsdisk/pagodo |
| **SearX** | 开源聚合搜索引擎，规避单引擎限流 | searx.github.io |

**GShark 部署示例**（Docker）：

```bash
docker run -d -p 8000:8000 \
  -e GITHUB_TOKEN=你的token \
  madneal/gshark
```

配置企业域名关键词后可持续监控新泄露。

---

## 七、防守视角：如何防止被 Dork

作为安全自查清单，企业应定期执行以下动作：

1. **robots.txt 不能作为安全手段**——Disallow 只引导爬虫，Dork 用户会直接访问；
2. **敏感目录与文件**：
   - 后台路径避免 `/admin`、`/login` 等可猜测命名；
   - 备份文件、配置文件禁止放在 Web 可访问目录；
   - 上传/生成的文档目录关闭目录列表；
3. **Robots 与索引控制**：
   - 管理后台设置 `X-Robots-Tag: noindex` 响应头；
   - 敏感系统使用独立域名并限制公网可达；
4. **定期自查**：用本手册的 Dork 对自有域名全量跑一遍，发现即处置；
5. **GitHub 治理**：
   - 建立 GitHub 组织并开启 secret scanning / push protection；
   - 定期用 GShark 监控企业域名关联泄露；
   - 员工入职培训强调代码脱敏；
6. **网盘泄露**：定期在网盘搜索引擎中以公司名为关键词排查公开分享；
7. **泄露处置**：联系搜索引擎提交移除请求，同时从源头删除文件并修改相关凭据。

---

## 八、参考资源

- [Google Hacking Database (GHDB) — Exploit-DB](https://www.exploit-db.com/google-hacking-database)
- [百度高级搜索语法官方帮助](https://help.baidu.com/question2?prod_id=5&class=509&id=3209)
- [AabyssZG/Open-Source-Information-Leakage — 开源项目信息泄露笔记](https://github.com/AabyssZG/Open-Source-Information-Leakage)
- [Google Dorking: A Goldmine for OSINT — OSINT Ambition](https://publication.osintambition.org)

---

> 📌 **持续更新**：本手册会随搜索引擎语法变化持续更新，欢迎提 Issue / PR 补充实战语法。
>
> ⭐ 如果本手册对你有帮助，欢迎 Star 支持！
