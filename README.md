# NetworkPanel / 青鸟云网络质量监测 — 项目说明文档

> 一个基于 **PHP 7.4 + MySQL 5.7 + Nginx** 的网络质量监测与测速工具站，提供在线测速、真实访客 IP 展示、IP 归属地/运营商查询、NAT 类型检测、流量消耗统计与排行榜、会员系统、后台管理等能力。前端为**原生 JavaScript（无前端框架）** + 服务端渲染页面，图表使用 ECharts，支持 PWA 面板设置（主题/紧凑/息屏常亮）。

---

## 目录

- [一、程序说明](#一程序说明)
  - [1.1 功能概述](#11-功能概述)
  - [1.2 使用方式](#12-使用方式)
  - [1.3 技术要点](#13-技术要点)
- [二、目录架构](#二目录架构)
- [三、数据库字典](#三数据库字典)
- [四、API 接口文档](#四api-接口文档)
- [五、部署指南](#五部署指南)
- [六、安全与上线注意事项](#六安全与上线注意事项)
- [七、常见问题](#七常见问题)

---

## 一、程序说明

### 1.1 功能概述

| 模块 | 入口 | 功能说明 |
|------|------|----------|
| **首页 / 控制台** | `index.php` | 网络测速（选节点、设线程、实时速度/流量/耗时）、IP 信息卡（服务端真实访客 IP + 公共出口双视图）、流量计算器、字节换算、数据源下拉切换、首页动态合规/公告模块 |
| **概览** | `home.php` | 站点概览介绍页（导航中「概览」指向此页） |
| **排行榜** | `rank.php` | 全网 / 今日流量消耗排名，领奖台样式展示前三名 |
| **NAT 检测** | `nat.php` | 基于 WebRTC + 多 STUN 节点探测，判定 NAT 类型（开放/锥型/对称型/未知）与映射行为 |
| **IP 工具** | `ip-tools.php` | IP 进阶查询（一致性、CDN、延迟、DNS 解析等） |
| **IP 查询** | `ip.php` | 单 IP 归属地、运营商、ASN 查询（带缓存） |
| **会员系统** | `user/*` | 注册 / 登录 / 个人中心 / 令牌管理 / 找回密码（已禁用）/ 社交登录（QQ、微信，开关可控） |
| **后台管理** | `admin/index.php` | 仪表盘、节点、会员、系统设置（基本/IP 设置/页脚/公告/广告）、操作日志、测速记录、用户反馈、页面管理、查询记录等 |
| **面板设置 / PWA** | `settings.php` | 主题（浅/深/随时段）、紧凑模式、息屏常亮（AOD）、导入导出配置 |

#### 后台管理功能详表（`admin/index.php` + `api/admin.php`）

| 模块 | 功能说明 |
|------|----------|
| 仪表盘 | 用户数、总流量、今日流量、节点数、未读反馈数；最新注册用户；最近操作日志 |
| 节点管理 | 测速节点 CRUD、分组、排序、HTTP/API 两种 URL 类型、启用/隐藏 |
| 会员管理 | 用户列表（分页/搜索/角色筛选）、编辑、重置密码、禁用/启用、详情（登录记录 + 流量统计） |
| 系统设置 | 基本设置、IP 设置（uapis / ip9 / xxapi 三家提供商手动切换）、系统配置（注册开关/线程限制/上报间隔）、页脚设置、公告/广告、页面管理、查询记录 |
| 操作日志 | 管理员增删改查审计，分页浏览 |
| 测速记录 | 所有用户测速详情（用户/节点/流量/耗时/速度/IP/UA/时间），搜索 + 分页 |
| 用户反馈 | 反馈列表（未读/已读/已处理筛选）、状态切换、批量清空 |
| IP 源配置 | `admin/ip_source.php` 单独管理生效 IP 源与 Token，保存后刷新 IP 缓存 |

### 1.2 使用方式

**最终用户**
1. 浏览器打开 `index.php` 即进入控制台，可立即开始测速、查看本机真实 IP 与归属地。
2. 顶部导航可在「首页 / 概览 / 工具箱 / NAT 检测 / 面板设置 / 排行榜 / 关于我们」间切换。
3. 注册 / 登录后可在个人中心查看累计流量、在线时长、访问令牌等。

**管理员**
1. 访问 `admin/index.php`，使用后台账号登录（默认 `admin` / `admin123`，**务必上线前修改**）。
2. 在「系统设置 → IP 设置」中切换 IP 提供商（uapis / ip9 / xxapi）；在「节点管理」增删测速节点；在「用户反馈」处理用户反馈。

**开发 / 部署**
- 本地开发环境为 Windows + PHPStudy（Nginx 1.15.11，PHP 7.4），**无 URL 重写**（入口即 `.php` 文件，前端路由靠 `#锚点`）。
- 项目自带回归自检：`tests/selfcheck.php`（含 320+ 项静态/契约断言），改动后建议执行。

### 1.3 技术要点

```
前端：
├── 原生 JavaScript（ES，无框架依赖，资源全本地、零 CDN）
├── ECharts（public/echarts.min.js，仅首页按需加载）
├── CSS 变量主题（--arsn-* / --np-*），支持浅/深/随时段
└── PWA（面板设置：主题/紧凑/AOD/导入导出）

后端：
├── PHP 7.4+（兼容 8.x），MySQL 5.7+（InnoDB，utf8mb4）
├── PDO 单例（getDB()），预处理语句防 SQL 注入，关闭模拟预处理 + 多语句
├── Session 认证（管理员 + 会员），会话 Cookie 显式 SameSite=Lax
├── password_hash()（Bcrypt）密码加密
└── 服务端统一 JSON 响应（jsonResponse()）

工程化：
├── 服务端渲染（render_page_header / layout.php 统一页头页尾）
├── IP 源可插拔注册表（ip_source_registry：uapis / ip9 / ipapi / ip_api_com / xxapi）
├── IP 查询带缓存（ip_cache 表，按 IP 维度）
└── 自动建表容错（关键表缺失时 API 内 CREATE TABLE IF NOT EXISTS 兜底）
```

---

## 二、目录架构

```
127.0.0.5/                          # 站点根目录（Web 根）
│
├── index.php                       # 🏠 首页 / 控制台（主入口，渲染测速面板 + IP 信息卡）
├── home.php                        # 📋 概览介绍页（导航「概览」指向此页）
├── rank.php                        # 🏆 流量消耗排行榜（领奖台）
├── nat.php                         # 🛰️ NAT 类型在线检测页（WebRTC + STUN）
├── ip.php                          # 🌐 IP 查询页
├── ip-tools.php                    # 🔧 IP 进阶工具页（一致性/CDN/延迟/DNS）
├── traffic.php                     # 🧮 流量计算器
├── byte.php                        # 🔢 字节换算工具
├── tools.php                       # 🧰 工具箱聚合页
├── settings.php                    # ⚙️ 面板设置 / PWA 配置页
├── about.php / agreement.php / privacy.php   # 关于 / 用户协议 / 隐私政策
├── redirect.php                    # 🔀 跳转 / 中转页（社交登录回调等）
│
├── config/
│   ├── config.php                  # 🔧 核心配置：DB 连接、jsonResponse()、getDB()、request_ip()/is_private_ip()、SameSite Cookie
│   └── cacert.pem                  # CA 证书包（curl SSL 验证用）
│
├── api/                            # 📡 接口目录（全部返回 JSON）
│   ├── admin.php                   # 后台核心 API（仪表盘/节点/用户/设置/日志/测速记录/反馈/页面/查询记录/公告/广告）
│   ├── admin_auth.php              # 后台管理员登录/登出/会话检查
│   ├── auth.php                    # 会员注册/登录/登出/资料/改密/反馈/令牌
│   ├── log.php                     # 流量日志上报写入（前台测速完成后调用）
│   ├── nodes.php                   # 前台可用节点列表
│   ├── ip.php                      # IP 查询（lookup/sources/registry/ipv6/latency）
│   ├── ip_advanced.php             # IP 进阶查询（一致性/CDN/延迟/DNS）
│   ├── nat.php                     # NAT 探测出口信息（仅返回出口 IP/地址族，不做 ICE 上传）
│   ├── rank.php / rank_period.php / rank_today.php  # 总榜 / 历史周期榜 / 今日榜
│   ├── query_log.php               # 前台查询记录（列表/删除/清空，服务端记录）
│   ├── page.php                    # 自定义页面（协议/隐私）增删改查
│   ├── tools.php                   # 工具类接口
│   ├── url_resolve.php             # NetworkPanel 协议 URL 解析
│   ├── footer_settings.php         # 页脚/公告/广告配置读取与首次建表
│   ├── speedtest.php               # 测速调度（ping/download，size 1–16MB 钳制）
│   ├── ucenter.php                 # UCenter 互通（与 Discuz! 等互通，签名校验）
│   └── ucenter.php / ...           # （见上）
│
├── admin/                          # 👤 后台管理
│   ├── index.php                   # 后台 SPA 页面 + 自包含引导脚本（登录/导航/设置表单/数据表格/弹窗开关）
│   ├── ip_source.php               # IP 源管理页（生效源下拉 + Token，保存后刷新缓存）
│   └── js/{utils,core,modules,content}.js  # 后台历史框架脚本（当前以 index.php 内联引导为主）
│
├── user/                           # 👥 用户中心页
│   ├── login.php / register.php / center.php / forgot.php / oauth.php
│
├── includes/                       # 🧩 公共层
│   ├── helpers.php                 # 转义/净化/限流/统一响应/http_get 等通用函数
│   ├── ip_service.php              # IP 源注册表 + ip_lookup / ip_fetch_remote（含缓存）
│   ├── layout.php                  # 页头页尾框架 + 全局导航 $navItems + 外观菜单
│   ├── pages.php / default_pages.php / page_view.php  # 协议/隐私等自定义页面
│   ├── site_header.php / site_footer.php / site_head.php  # 站点片段
│   ├── auth_head.php / user_head.php  # 认证/用户中心头部
│   ├── query_log.php               # 查询记录服务端直记 + 前端 window.reportQuery
│   ├── units.php                   # 单位换算（字节/速率）
│   └── UCenter.php                 # UCenter 客户端（@unserialize 外部响应，建议开启签名校验）
│
├── css/                            # 🎨 样式
│   ├── style.css / console.css / home.css / ip.css / nat.css / rank.css / panel.css
│   ├── admin.css / auth.css / aod.css / classic.css / pages.css
│
├── js/                             # 📜 前台 JS
│   ├── app.js                      # 首页主逻辑（IP 信息卡/数据源切换/图表/测速）
│   ├── home-modules.js / home.js / ip.js / ip-tools.js / nat.js / rank.js
│   ├── settings.js / theme.js / traffic.js / byte.js / layout.js / aod.js
│   └── auth-*.js（login/register/forgot/oauth/common）
│
├── public/                         # 🖼️ 静态资源
│   ├── echarts.min.js              # 图表库（首页按需加载）
│   └── favicon.svg
│
├── database/                       # 💾 数据库
│   ├── schema.sql                  # 完整建表 + 默认数据（15 张表，幂等 INSERT IGNORE）
│   └── upgrade_*.sql               # 增量升级脚本（oauth/pages/query_log/thread_max_16/ucenter）
│
├── tests/
│   └── selfcheck.php               # 项目回归自检（静态/契约断言，320+ 项）
│
├── .htaccess / nginx.htaccess      # 服务器配置参考（Apache / Nginx 伪静态）
└── README.md                       # 本文档
```

> 说明：站点运行仅依赖上述「核心源码 + 必要配置」。开发期交付笔记（`CONSOLE_THEME_NOTES.md` 等）与孤立备份（`admin/admin.js.bak_*`）、临时残留文件（`nul` 等）已从根目录清理，详见 [第六节](#六安全与上线注意事项)。

---

## 三、数据库字典

数据库名：`netart_czkree_co`（由 `config.php` 的 `DB_NAME` 定义），字符集 `utf8mb4`，存储引擎 InnoDB。
共 **15 张表**，按业务分为：核心业务表、统计聚合表、配置/日志表。

### 表清单一览

| 表名 | 用途 | 量级 |
|------|------|------|
| `users` | 会员 / 用户表 | 中 |
| `admins` | 后台管理员表 | 极小 |
| `nodes` | 测速节点表 | 小 |
| `site_settings` | 网站配置键值对表 | 小 |
| `traffic_logs` | 测速流量日志表 | 大 |
| `query_logs` | 前台查询记录表（行为溯源） | 大 |
| `ip_cache` | IP 查询缓存表 | 大 |
| `user_feedback` | 用户反馈表 | 小 |
| `announcements` | 公告管理表 | 小 |
| `ads` | 广告管理表 | 小 |
| `admin_logs` | 管理员操作审计日志 | 中 |
| `stats_hourly` / `stats_daily` / `stats_monthly` / `stats_yearly` | 流量统计聚合表（4 张同构） | 中/小 |

---

### 3.1 users — 会员 / 用户表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 用户 ID |
| `username` | VARCHAR(50) | UNIQUE, NOT NULL | - | 用户名 |
| `email` | VARCHAR(100) | | '' | 邮箱（可登录） |
| `password_hash` | VARCHAR(255) | NOT NULL | - | Bcrypt 密码哈希 |
| `access_token` | VARCHAR(64) | UNIQUE, NOT NULL | - | API 访问令牌（64 位十六进制） |
| `headimg` | VARCHAR(500) | | '' | 头像 URL |
| `isp` | VARCHAR(50) | | '' | 运营商（移动/电信/联通…） |
| `region` | VARCHAR(100) | | '' | 地区（如「北京市」） |
| `region_short` | VARCHAR(100) | | '' | 地区简称（如「北京」） |
| `ip` | VARCHAR(45) | | '' | 最后登录 IP |
| `role` | ENUM('user','vip','admin') | | 'user' | 角色：普通 / VIP / 管理员 |
| `status` | TINYINT | | 1 | 0=禁用 1=正常 |
| `total_bytes` | BIGINT UNSIGNED | | 0 | 累计使用流量（字节），由 `log.php` 实时累加 |
| `total_time` | DECIMAL(12,2) | | 0 | 累计在线时长（秒） |
| `last_active` | DATETIME | | NULL | 最后活跃时间 |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 注册时间 |
| `updated_at` | DATETIME | | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：** `idx_role`, `idx_status`, `idx_access_token`

### 3.2 admins — 后台管理员表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 管理 ID |
| `username` | VARCHAR(50) | UNIQUE, NOT NULL | - | 登录账号 |
| `password_hash` | VARCHAR(255) | NOT NULL | - | Bcrypt 密码哈希 |
| `nickname` | VARCHAR(50) | | '' | 显示昵称 |
| `role` | ENUM('super_admin','admin','editor') | | 'admin' | 权限等级 |
| `last_login` | DATETIME | | NULL | 上次登录时间 |
| `last_ip` | VARCHAR(45) | | '' | 上次登录 IP |
| `login_count` | INT UNSIGNED | | 0 | 登录次数 |
| `status` | TINYINT | | 1 | 0=禁用 1=正常 |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 创建时间 |

**默认账号：** `admin` / `admin123`（请上线前修改！）

### 3.3 nodes — 测速节点表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 节点 ID |
| `group_name` | VARCHAR(50) | NOT NULL | '默认' | 分组（如「运营商」「全球[Global]」） |
| `label` | VARCHAR(100) | NOT NULL | - | 节点显示名称 |
| `url` | VARCHAR(500) | NOT NULL | - | 测速目标 URL（http 直链或 api 协议地址） |
| `url_type` | ENUM('http','api') | | 'http' | http=直接下载测速；api=协议解析后多线程 |
| `sort_order` | INT | | 0 | 排序权重（越小越靠前） |
| `status` | TINYINT | | 1 | 0=隐藏 1=启用 |
| `description` | VARCHAR(255) | | '' | 备注描述 |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 创建时间 |
| `updated_at` | DATETIME | | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：** `idx_group`, `idx_status`, `idx_sort`

### 3.4 site_settings — 网站配置表（键值对）

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 配置项 ID |
| `setting_key` | VARCHAR(80) | UNIQUE, NOT NULL | - | 配置键名 |
| `setting_value` | TEXT | | NULL | 配置值 |
| `setting_type` | ENUM('string','int','bool','json','text') | | 'string' | 值类型（决定后台输入控件） |
| `title` | VARCHAR(100) | | '' | 中文标题（后台显示） |
| `description` | VARCHAR(255) | | '' | 说明文字 |
| `group_name` | VARCHAR(50) | | 'basic' | 分组：basic/system/footer/header/contact/content/homepage/oauth/ucenter/pages/user_center/advanced |
| `sort_order` | INT | | 0 | 同组内排序 |
| `updated_at` | DATETIME | | CURRENT_TIMESTAMP ON UPDATE | 最后修改时间 |

**主要配置项：**

| setting_key | 分组 | 类型 | 说明 |
|-------------|------|------|------|
| `site_name` | basic | string | 网站名称 |
| `site_description` | basic | text | 网站描述（SEO） |
| `site_logo` | basic | string | Logo URL |
| `banner_url` / `banner_link` | basic | string | 顶部横幅图 / 跳转链接 |
| `traffic_price` / `traffic_default_speed` | basic | string | 流量计算器单价(元/GB) / 默认速率(Mbps) |
| `ip_source_active` | basic | string | 生效 IP 源（uapis/ip9/xxapi…） |
| `ip_source_uapis_token` | basic | string | UAPIS 接口 Token |
| `register_open` | system | bool | 是否开放注册 |
| `default_thread_num` / `max_thread_num` / `log_interval` | advanced | int | 默认线程 / 最大线程 / 上报间隔(秒) |
| `footer_copyright` / `footer_icp` / `footer_icp_url` | footer | string | 版权 / ICP 备案号 / 备案链接 |
| `friend_links` | footer | json | 友情链接 JSON 数组 |
| `notices` / `footer_html` | footer | text | 公告（多行）/ 自定义 HTML |
| `nav_links` | header | json | 顶部导航链接 JSON 数组 |
| `contact_qq` / `contact_qq_group` | contact | string | 联系 QQ / QQ 群 |
| `about_content` | content | json | 关于弹窗内容 |
| `page_agreement` / `page_privacy` | pages | text | 用户协议 / 隐私政策正文 |
| `home_compliance_enabled` / `home_compliance_text` / `home_compliance_bg` | homepage | bool/text/string | 首页合规提示开关 / 内容 / 底色 |
| `home_notice_enabled` / `home_notice_text` | homepage | bool/text | 首页公告提示开关 / 内容 |
| `uc_card_enabled` / `uc_card_text` / `uc_card_url` | user_center | bool/string/string | 会员中心自定义卡片 |

> `oauth` / `ucenter` 分组用于社交登录与 UCenter 互通开关（配置驱动，无独立持久表）。

### 3.5 traffic_logs — 测速流量日志表（核心业务表）

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | BIGINT UNSIGNED | PK, AUTO_INCREMENT | - | 日志 ID |
| `user_id` | INT UNSIGNED | | 0 | 用户 ID（0=匿名/游客） |
| `url` | VARCHAR(500) | NOT NULL | '' | 测速使用的节点 URL |
| `thread_num` | TINYINT UNSIGNED | | 1 | 使用的线程数 |
| `bytes_used` | BIGINT UNSIGNED | | 0 | 本次消耗字节数 |
| `duration` | DECIMAL(10,2) | | 0 | 本次耗时（秒） |
| `speed` | BIGINT UNSIGNED | | 0 | 平均速度（字节/秒） |
| `ip` | VARCHAR(45) | | '' | 客户端真实 IP |
| `user_agent` | VARCHAR(500) | | '' | 浏览器 UA |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 测试开始时间 |

**索引：** `idx_user_id`, `idx_created_at`, `idx_bytes_used`
**写入时机：** 前台每次测速完成后由 `api/log.php` 写入，并联动更新 `users.total_bytes/total_time` 与 `stats_*`。

### 3.6 query_logs — 前台查询记录表（行为溯源）

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | BIGINT UNSIGNED | PK, AUTO_INCREMENT | - | 记录 ID |
| `user_id` | INT UNSIGNED | | 0 | 用户 ID（0=游客） |
| `username` | VARCHAR(64) | NOT NULL | '' | 用户名或游客标识 |
| `query_type` | VARCHAR(32) | NOT NULL | '' | 查询类型：ip / ip_advanced / traffic / byte / speedtest / ip_tools |
| `query_input` | VARCHAR(1000) | NOT NULL | '' | 查询输入（目标 IP/域名/参数） |
| `result_summary` | VARCHAR(500) | NOT NULL | '' | 结果摘要 |
| `ip` | VARCHAR(45) | | '' | 客户端 IP |
| `user_agent` | VARCHAR(500) | | '' | 浏览器 UA |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 查询时间 |

**索引：** `idx_user_id`, `idx_query_type`, `idx_created_at`, `idx_ip`

### 3.7 ip_cache — IP 查询缓存表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 缓存 ID |
| `ip` | VARCHAR(45) | UNIQUE, NOT NULL | - | IP 地址（唯一键） |
| `country_code` | CHAR(2) | | '' | 国家代码（CN/US…） |
| `country_name` | VARCHAR(100) | | '' | 国家全称 |
| `registered_country_code` | CHAR(2) | | '' | 注册国家代码（与归属国区分） |
| `registered_country_name` | VARCHAR(100) | | '' | 注册国家全称 |
| `regions` | JSON | | NULL | 地区层级数组 |
| `regions_short` | JSON | | NULL | 地区简称数组 |
| `asn_number` | INT UNSIGNED | | 0 | ASN 编号 |
| `asn_name` | VARCHAR(255) | | '' | ASN 组织名 |
| `asn_info` | VARCHAR(255) | | '' | ASN 附加信息 |
| `type` | VARCHAR(20) | | '' | IP 类型（如 IPv4/IPv6） |
| `addr` | VARCHAR(255) | | '' | 归属地详细描述 |
| `cached_at` | DATETIME | | CURRENT_TIMESTAMP | 缓存时间 |

**索引：** `idx_cached_at`

### 3.8 user_feedback — 用户反馈表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 反馈 ID |
| `user_id` | INT UNSIGNED | NOT NULL | - | 提交用户 ID |
| `username` | VARCHAR(50) | NOT NULL | '' | 用户名（冗余，避免 JOIN） |
| `content` | TEXT | NOT NULL | - | 反馈内容（最多 500 字） |
| `status` | TINYINT | | 0 | 0=未读 1=已读 2=已处理 |
| `reply` | TEXT | | NULL | 管理员回复（仅后台可见） |
| `ip` | VARCHAR(45) | | '' | 提交者 IP |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 提交时间 |
| `replied_at` | DATETIME | | NULL | 回复时间 |

**索引：** `idx_user_id`, `idx_status`, `idx_created_at`

### 3.9 announcements — 公告管理表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 公告 ID |
| `title` | VARCHAR(100) | | '公告' | 公告标题 |
| `content` | TEXT | NOT NULL | - | 公告内容（支持 HTML） |
| `status` | TINYINT | | 1 | 0=禁用 1=启用 |
| `sort_order` | INT | | 0 | 排序权重（越小越前） |
| `show_popup` | TINYINT | | 1 | 是否弹窗展示（0否 1是） |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 创建时间 |
| `updated_at` | DATETIME | | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

### 3.10 ads — 广告管理表

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | INT UNSIGNED | PK, AUTO_INCREMENT | - | 广告 ID |
| `title` | VARCHAR(200) | NOT NULL | - | 广告文字 |
| `url` | VARCHAR(500) | NOT NULL | - | 跳转链接 |
| `color` | VARCHAR(20) | | '' | 文字颜色（空=默认） |
| `status` | TINYINT | | 1 | 0=禁用 1=启用 |
| `sort_order` | INT | | 0 | 排序权重 |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 创建时间 |
| `updated_at` | DATETIME | | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

### 3.11 admin_logs — 管理员操作审计日志

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| `id` | BIGINT UNSIGNED | PK, AUTO_INCREMENT | - | 日志 ID |
| `admin_id` | INT UNSIGNED | NOT NULL | - | 操作管理员 ID |
| `action` | VARCHAR(50) | NOT NULL | - | 操作类型（create/update/delete/disable/login/logout…） |
| `target_type` | VARCHAR(30) | | '' | 目标类型（user/node/setting/feedback…） |
| `target_id` | INT UNSIGNED | | 0 | 目标对象 ID |
| `detail` | TEXT | | NULL | 操作详情（通常为 JSON） |
| `ip` | VARCHAR(45) | | '' | 操作者 IP |
| `created_at` | DATETIME | | CURRENT_TIMESTAMP | 操作时间 |

**索引：** `idx_admin`, `idx_action`, `idx_created`

### 3.12 stats_hourly / stats_daily / stats_monthly / stats_yearly — 流量统计聚合表（4 张同构）

按时间维度分表，使用 `UPSERT`（INSERT … ON DUPLICATE KEY UPDATE）累加：

| 表名 | 维度 | 唯一键 |
|------|------|-----------|
| `stats_hourly` | 小时 | `(user_id, hour_start)` |
| `stats_daily` | 日 | `(user_id, day_start)` |
| `stats_monthly` | 月 | `(user_id, month_start)` |
| `stats_yearly` | 年 | `(user_id, year)` |

**公共字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | BIGINT UNSIGNED | PK 自增 |
| `user_id` | INT UNSIGNED | 用户 ID（0=全局汇总） |
| `{period}_start` | DATETIME / DATE / SMALLINT | 时间起点（year 为 SMALLINT 年份） |
| `total_bytes` | BIGINT UNSIGNED | 累计字节数（UPSERT 累加） |
| `total_duration` | DECIMAL(12,2) | 累计时长秒数（UPSERT 累加） |
| `avg_speed` | DECIMAL(15,2) | 平均速度（自动重算） |

---

## 四、API 接口文档

> 所有接口返回 JSON；后台接口需管理员登录态（`$_SESSION['admin_id']`）。

### 后台核心 API（`api/admin.php`）

| Method | Action | 参数 | 说明 |
|--------|--------|------|------|
| GET | `dashboard` | - | 仪表盘统计 + 趋势 + 最新用户 + 日志 |
| GET/POST | `nodes` / `node_add` / `node_edit` / `node_delete` | 见后台表单 | 节点 CRUD |
| GET | `users` | page, limit, search, role? | 会员列表 |
| GET/POST | `user_detail` / `user_update` / `user_delete` | id, … | 会员详情/编辑/禁用 |
| GET | `settings` | group? | 获取配置（按分组） |
| POST | `setting_save` | key, value | 保存单个配置项 |
| GET | `logs` | page, limit | 操作日志 |
| GET | `speed_logs` | page, limit, keyword? | 测速记录（JOIN users，可搜索） |
| GET | `feedback_list` | page, limit, status? | 用户反馈列表（status 可空，安全读取） |
| POST | `feedback_set_status` | id, status(0/1/2) | 设置反馈状态 |
| POST | `feedback_clear` | hard_reset? | 清空所有反馈 |
| GET/POST | `announcements` / `ads` | 见后台表单 | 公告 / 广告 CRUD |
| GET/POST | `page_list` / `page_save` / `page_reset` | key, content | 自定义页面（协议/隐私） |
| GET/POST | `query_log_list` / `query_log_delete` / `query_log_clear` | page, limit, id? | 前台查询记录管理 |

### 会员 API（`api/auth.php`）

| Method | Action | 参数 | 说明 |
|--------|--------|------|------|
| POST | `register` | username, password, email | 注册（受 `register_open` 控制） |
| POST | `login` | username, password | 登录（Session + token） |
| POST | `logout` | - | 登出 |
| GET | `profile` | - | 当前用户信息 |
| POST | `update` | email?/headimg?/new_password?/old_password? | 修改资料 |
| POST | `reset_stat` | field(total_bytes/total_time) | 重置个人统计 |
| POST | `refresh_token` | - | 重新生成 access_token |
| POST | `submit_feedback` | content | 提交反馈 |

### 流量上报（`api/log.php`）

**Method:** POST（JSON Body）

```json
{
  "AccessToken": "用户64位访问令牌",
  "url": "测速节点URL",
  "threadNum": 8,
  "used": 104857600,
  "time": 12.5
}
```

**响应：** `{"status":0,"message":"日志记录成功"}`
**副作用：** 写 `traffic_logs` → 更新 `users.total_bytes/total_time` → UPSERT `stats_hourly/daily/monthly/yearly`。

### IP 查询（`api/ip.php`）

| Action | 参数 | 说明 |
|--------|------|------|
| `lookup` | ip?, source? | IP 归属/运营商/ASN（source 可临时覆盖生效源） |
| `sources` | - | 列出已配置的源 |
| `registry` | - | 列出全部可用源 + 当前生效源（前台下拉用） |
| `ipv6` | - | 本机 IPv6 检测 |
| `latency` | - | 多目标延迟探测 |

---

## 五、部署指南

### 环境要求
- PHP ≥ 7.4（推荐 8.0+），需 `pdo_mysql`、`curl`、`openssl` 扩展
- MySQL ≥ 5.7 / MariaDB ≥ 10.3（InnoDB，utf8mb4）
- Web 服务器：Nginx（本项目基于 PHPStudy，无 URL 重写）或 Apache

### 安装步骤
1. **上传文件**：将站点根目录全部文件上传至 Web 根。
2. **建库建表**：导入 `database/schema.sql`（幂等，可重复执行），并按需执行 `upgrade_*.sql`。
3. **改配置**：编辑 `config/config.php`，填入真实数据库信息（`DB_NAME` 默认 `netart_czkree_co`，上线请改为独立库名）。
4. **改默认密码**：`UPDATE admins SET password_hash='$2y$10$新bcrypt哈希' WHERE username='admin';`
5. **配置服务器**：Nginx 参考 `nginx.htaccess`；Apache 启用 `mod_rewrite` 并放置 `.htaccess`。
6. **访问**：前台 `index.php`；后台 `admin/index.php`（默认 admin/admin123）。

---

## 六、安全与上线注意事项

本项目已通过一轮代码审查与加固，**生产部署前请逐项确认**：

1. **已修复的安全阻断项（上线前务必复核）**
   - Web 根目录的未授权改密工具 `reset_password.php` 与数据库导出 SQL 文件已从根目录移出（备份于 `D:\wangzhan\_removed_from_webroot\`），**请勿重新放回 Web 根**。
   - `api/admin.php`、`api/auth.php` 的异常/致命错误处理器已改为**仅服务端 `error_log` + 返回通用提示**，不再向客户端泄露文件路径 / SQL / 栈信息。
   - `api/admin.php::hasColumn()` 已加标识符白名单校验，关闭表名/列名注入面。

2. **上线前建议完成**
   - **修改数据库账号**：`config.php` 中 `root/123456` 为本地开发凭据，上线应改为**低权限专用账号 + 强密码**（并同步改库）。
   - **收紧 CORS**：当前 `jsonResponse()` 使用 `Access-Control-Allow-Origin: *`；含用户数据的接口建议限制来源。
   - **CSRF 防护**：状态变更接口目前仅依赖会话 Cookie 的 `SameSite=Lax` 作为纵深缓解，高安全场景应补充 CSRF Token。
   - **默认管理员密码**：务必修改 `admin/admin123`。
   - **目录整洁**：开发期交付笔记（`CONSOLE_THEME_NOTES.md`、`NAT_DETECTION_NOTES.md`、`PANEL_SETTINGS_NOTES.md`、`REFACTOR_REPORT.md`）与孤立备份（`admin/admin.js.bak_2026-09-10`）、Windows 残留空文件（`nul`）已从根目录清理，保持仅含核心源码与必要配置。
   - **UCenter 集成**：`includes/UCenter.php` 对外部响应使用 `@unserialize()`，若启用 UCenter 互通，请确保其签名校验已开启。

---

## 七、常见问题

### Q: 后台测速记录一直显示「加载中…」？
**A:** ① 按 F12 看控制台是否有 JS 报错；② 直接访问 `api/admin.php?action=speed_logs&page=1&limit=20` 看 API 返回；③ `SELECT COUNT(*) FROM traffic_logs;` 检查表与数据；④ 表缺失时执行 `database/schema.sql` 中的建表语句（API 也会自动建表兜底）。

### Q: 修改用户密码后前台无法登录？
**A:** 确认密码字段是 `password_hash`（非 `password`），后台 `user_update` 写入该列。若列名写错，需后台重新保存一次密码。

### Q: 排行榜没有数据？
**A:** 排行榜读取 `users.total_bytes`，需有用户测速上报过 `api/log.php` 才会有数据。可手动：`UPDATE users SET total_bytes=104857600 WHERE username='test';`

### Q: 如何切换 IP 提供商（uapis / ip9 / xxapi）？
**A:** 后台 → 系统设置 → IP 设置，下拉选择生效源并保存（如选 uapis 需填 Token）；保存后系统自动清空 `ip_cache` 使新源立即生效。前台首页 IP 面板也可通过数据源下拉临时切换（仅本次请求生效，不改全局）。

### Q: 注册功能如何关闭？
**A:** 后台 → 系统设置 → 找到「开放注册」设为「否」，或 `UPDATE site_settings SET setting_value='0' WHERE setting_key='register_open';`

### Q: 根目录为什么没有 `reset_password.php`？
**A:** 该文件为未授权改密工具，属安全隐患，已从 Web 根移除（备份在 `_removed_from_webroot`）。密码重置请直接操作数据库或后台会员管理。
