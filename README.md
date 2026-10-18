# Vultr搭建WordPress：一键部署和手动安装怎么选，从套餐价格到 SSL 证书的完整步骤

在 Vultr 上搭 WordPress，绝大多数人选官方 Marketplace 的 WordPress 一键应用就够了：服务器约 60 秒部署完成，Nginx、PHP、MariaDB、防火墙和 SSL 工具全部预先配好，剩下的事只有解析域名、签一张免费证书、跑完安装向导。想在干净系统上手动装 LEMP 也行，Vultr 官方文档有现成路线，只是时间成本高不少。

费用先给个数：**$6/月起，按小时计费、每月封顶**，服务器随时可以销毁。唯一建议下单前想清楚的是速度——Vultr 的线路没有针对中国大陆优化，访客主要在大陆的话体验可能一般，后面单独说。

> **选购速览**：个人博客或企业官网，选 High Performance AMD 的 **$6 档**（1 核 / 1 GB / 25 GB NVMe）；要跑 WooCommerce 或访客量上来之后，换 **$24 档**（2 核 / 4 GB / 100 GB NVMe）。👉 [查看 Vultr 全部套餐与当前价格](https://www.vultr.com/pricing/?ref=7268283)

## Vultr 搭 WordPress 有两种方式，差别在哪

一键应用和手动安装，区别只在于你要不要控制服务器本身。

一键应用把 WordPress 需要的整套环境都打包好了：Nginx、PHP、MySQL/MariaDB、WordPress、phpMyAdmin，外加 Wordfence 安全插件、SMTP Mailer 发信插件和 Cockpit 服务器面板，开机即用。Marketplace 里还有个 OpenLiteSpeed 版本，LiteSpeed 的缓存对高流量的 WooCommerce 店有吸引力，但第一次装，选标准版就行。

手动安装是在干净的 Ubuntu 或 Debian 上自己装 LEMP，Vultr 官方文档给的是 Ubuntu 24.04 LTS 路线。它适合这几种情况：需要自己指定 PHP 版本、想在一台服务器上跑多个 WordPress 站点、或者要改 Nginx 配置做优化——这些在一键版里都被锁死了。

说白了，如果你只是想把 WordPress 跑起来写文章，手动安装省下的那点灵活度，抵不上多花的几个小时。

## 套餐价格与配置：搭 WordPress 看这几档就够

主流选择就两档：$6 起步，$24 上强度。以下是 Vultr 搭 WordPress 最常用到的四档：

| 套餐 | vCPU / 内存 | 存储 | 月付价格 | 购买入口 |
| --- | --- | --- | --- | --- |
| Regular Compute | 1 核 / 1 GB | 25 GB SATA SSD | $5/月 | [ 开通 $5 套餐](https://www.vultr.com/pricing/?ref=7268283) |
| High Performance AMD | 1 核 / 1 GB | 25 GB NVMe | $6/月 | [ 开通 $6 套餐](https://www.vultr.com/pricing/?ref=7268283) |
| High Performance AMD | 2 核 / 4 GB | 100 GB NVMe | $24/月 | [ 开通 $24 套餐](https://www.vultr.com/pricing/?ref=7268283) |
| Optimized General Purpose GP-1 | 1 核 / 4 GB | 30 GB NVMe | $30/月 | [ 开通 GP-1 套餐](https://www.vultr.com/pricing/?ref=7268283) |

除了表格里的配置，Vultr 还有更多 vCPU 和内存组合、专用 CPU 机型，部署时按需选择；想要 2 GB 内存，也有 1 核 2 GB 的 $10 档。对绝大多数 WordPress 站点，上表四档基本就是决策范围。价格最近一次核对是 2026 年 6 月，下单时以 Vultr 页面显示为准。

$6 档的卖点是 AMD EPYC 处理器加 NVMe 盘——WordPress 每次处理页面请求都在频繁读写磁盘，NVMe 的随机读写优势很实在。Better Stack 做了为期 12 个月的基准测试，这一档在纽约的平均 TTFB 约 145ms，伦敦约 185ms，对一个 1 GB 内存的小机器来说算体面。$24 档在 Geekbench 6 里单核 1926 分、多核 3513 分，对 PHP 来说单核成绩最要紧。

### 真实成本：按小时计费意味着什么

Vultr 按小时扣费，封顶月价。$24 档约每小时 $0.036，开一整天测试也就八毛多美元，觉得不合适就销毁，计费即停。这套模式很适合先开一台试试再决定留不留。

算一笔第一年的账：服务器 $6 × 12 = $72；域名另算，价格看注册商；SSL 证书免费，certbot 已经预装在系统里；自动备份是付费功能，不开的话用 WordPress 备份插件自己定时备。参照系是托管型 WordPress 主机，通常每月 $25–50，Vultr 便宜一个量级，代价是运维自己来。

付款方式上，信用卡和 PayPal 都行，支付宝也支持，微信支付已经下线。新用户时不时能领到限时试用额度，金额和条件随活动变化，一般要求信用卡或 PayPal，有效期也有限。👉 [先按小时开一台 Vultr，测完再决定留不留](https://bit.ly/VulTR)

## 一键部署 WordPress 全流程

### 第一步：部署服务器

登录 Vultr 后，点 Products → Deploy New Server，Choose Image 选 Marketplace Apps，搜 WordPress。机房有 32 个，分布六大洲，价格统一——按访客的位置选，别按自己的位置选。套餐选好之后，建议顺手加上 SSH key，比纯密码登录安全。点 Deploy Now，大约一分钟开好，然后到 Server Information 页面记下 root 密码和 IP。👉 [打开 Vultr 的 WordPress 一键应用页面](https://www.vultr.com/marketplace/apps/wordpress/?ref=7268283)

### 第二步：把域名解析过来

在域名服务商那里加一条 A 记录，指向服务器 IP。生效一般几分钟，最长可能要等 48 小时。可以用在线工具（比如 dnschecker.org）确认全球解析已经生效，再进行下一步。

### 第三步：签一张免费 SSL 证书（建议现在就做）

SSH 以 root 登录服务器，certbot 已经预装，一条命令搞定：


certbot --nginx --redirect -d www.example.com -d example.com -m admin@example.com --agree-tos --no-eff-email


把域名和邮箱换成你自己的。`-d` 后面写两个地址——裸域 example.com 和 www.example.com，一条命令签出多域名证书；`--redirect` 会让 Nginx 自动把 HTTP 请求跳到 HTTPS。

为什么建议现在装：WordPress 会把完整 URL 写进数据库，如果先跳过 HTTPS、以后再补，之前发布的 http:// 链接全部要靠插件修正。现在花一分钟，省掉以后两小时的坑。

### 第四步：wp-admin 之前的那个登录弹窗

访问 `https://你的域名/wp-admin/`，浏览器会先弹一个"登录"框。这是服务器层面的安全验证，保护你的站点在 WordPress 还没装完时不被乱入——输入 Server Information 页面里的 security user 账号密码，不是你的 WordPress 密码。装完之后可以在后台把这个弹窗关掉。

没买域名、直接用 IP 访问的话，会遇到自签证书的安全警告，点"高级 → 继续访问"就能过，不影响安装，但长期用还是买个域名省心。

### 第五步：跑安装向导，把域名写进后台

向导里选语言、填站点标题、管理员用户名、邮箱和密码，系统生成的强密码记得存下来。装完先去 Settings → General，把 WordPress Address (URL) 和 Site Address (URL) 都改成你的域名。

这里的顺序很重要：**先改域名，再发内容**。WordPress 把带域名的完整链接存在数据库里，如果先用 IP 地址发了一批文章再改，旧文章里的图片和内链全都会指错地方。真踩了这个坑，先备份数据库，然后用搜索替换类插件批量修正。

### 预装组件里，建议马上启用这两个

Wordfence 是端点防火墙加恶意软件扫描，在 Plugins → Installed Plugins 里激活就行，多一层日志和扫描不亏。

SMTP Mailer 解决的是发信问题——VPS 上的 WordPress 默认发邮件经常不可靠，注册验证码、密码重置邮件发不出去是常事。激活插件后配置你的 SMTP 服务即可。

另外 phpMyAdmin 和 Cockpit 面板（9080 端口）也在，Cockpit 的账号密码在 Server Information 页面；如果你启用了 Vultr 防火墙，记得放行 9080 端口。

## 手动安装 LEMP：什么人值得折腾

手动路线适合明确知道自己要什么的人：一台服务器跑多个站点、锁定特定 PHP 版本、对 Nginx 配置有想法。系统推荐 Ubuntu 24.04 LTS（Vultr 官方文档这条路线 2026 年 4 月刚更新过）或 Debian 12。

流程概览：更新系统 → 安装 Nginx 并放行防火墙 → 装 MariaDB 建数据库 → 装 PHP 及常用扩展 → 从 wordpress.org 下载程序 → 配置 wp-config.php 和虚拟主机 → 用 certbot 上证书。环境版本尽量对齐 WordPress 官方建议（PHP 8.3+、MySQL 8.0+ 或 MariaDB 10.6+），新版对内存要求也更高，1 GB 内存的机器别同时开太多服务。

这条路线真正的成本在第一次搭建和之后每次安全更新、性能调优。值不值，取决于你想在这台服务器上学多少东西。

## 中国大陆访客的速度问题，下单前想清楚

Vultr 的线路没有针对中国大陆优化，大陆访客直连的速度一般，这是中文用户下单前最值得权衡的一点。面向欧美访客的站点随便选机房，32 个位置同价；读者主要在大陆的话，要么接受一般速度，要么在前面套一层 CDN，要么考虑主打 CN2 这类大陆优化线路的商家。

一个常见误区提醒：测速时别只测自己电脑的打开速度。你的访客在哪里，服务器就该选在哪里——你在国内用代理测出的流畅度，代表不了你读者的体验。

## 几个高频问题

### 不买域名能搭 WordPress 吗？

能。用 IP 地址照样跑完安装向导，但自签证书警告会一直在，HTTPS 那一步也做不了，浏览器体验很差。域名注册只要几分钟，建议先解析再装。

### $5 档真的够用吗？

纯文字博客跑得动。但它是 SATA SSD 加共享 CPU，一旦加缓存插件或 WooCommerce 扩展，体验会明显打折。$5 和 $6 之间差一块钱，换来 NVMe 盘和更好的处理器，多数情况下这钱不省。

### 费用怎么算，能随时停吗？

按小时计费、每月封顶，销毁服务器即停止计费，没有长期合约。想验证某个配置够不够用，开一天试试再决定，成本不到一美元。

### 能跑 WooCommerce 吗？

能，一键应用走的是同一套流程。配置建议直接上 $24 档，2 核 4 GB 加 100 GB NVMe，Better Stack 的 Geekbench 6 测试里它的单核成绩约为 DigitalOcean Basic Droplet 的 2.5 倍，而这个指标恰恰最影响 PHP 响应速度。

## 按你的情况做决定

- 个人博客、公司官网：$6 的 High Performance AMD 档加一键应用，装完顺手跑 certbot，十分钟收工。
- WooCommerce 或流量在涨：$24 档起步，免得三个月后搬家。
- 想学 Linux 运维、要多站点：手动 LEMP，Ubuntu 24.04 或 Debian 12 都行。
- 访客主要在中国大陆：先想清楚速度预期，别急着下单。

下单前确认三件事：域名已注册、A 记录准备好、机房按访客位置选。SSH key 记得在部署时加上。都齐了的话，👉 [去 Vultr 把你的第一台 WordPress 服务器跑起来](https://bit.ly/VulTR)
