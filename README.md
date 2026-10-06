# 📡 Collect-IPTV

自动收集互联网上公开的 IPTV 节目源，借助 GitHub Actions 检测可用性与延迟，去重后为每个频道保留延迟最低的一条地址，生成带分组和台标的 M3U 播放列表，每 4 小时自动更新一次。

本仓库 fork 自 [zilong7728/Collect-IPTV](https://github.com/zilong7728/Collect-IPTV)，在其基础上更换了台标图床并增加了台标名规范化。

> ⚠️ **特别说明：** 检测在 GitHub 服务器上进行，**不保证国内网络环境下的链接速度与可用性**。  
> ⚠️ **部分节目源来自互联网公开资源，上游源可能自行添加广告、弹窗、二维码或“赞助”提示。此类内容由上游节目源提供，与本项目及项目作者本人无关，本项目未主动添加、推广或收取相关赞助。**  
> ⚠️ 所有频道的完整性与有效性高度依赖上游网络资源；若上游频道源大面积失效，自动更新时可能会被可用性检测过滤。

---

## 🔗 订阅地址

<!-- Generated File Link M3U --> [View M3U File](https://raw.githubusercontent.com/shocklele/Collect-IPTV/refs/heads/main/best_sorted.m3u)

<!-- Generated File Link M3U8 --> [View M3U8 File](https://raw.githubusercontent.com/shocklele/Collect-IPTV/refs/heads/main/best_sorted.m3u8)

两个文件内容相同，只是扩展名不同，按播放器支持的格式任选其一。

## ⏱️ 最近更新时间

<!-- Last Run Time --> 2026-10-06 15:55:14 CST

## 💡 使用说明

1. 复制上方订阅地址，或下载对应的 M3U / M3U8 文件
2. 导入支持 IPTV 的播放器（如 Kodi、PotPlayer、Perfect Player、APTV 等）
3. 节目源每 4 小时自动更新，使用订阅地址的播放器刷新即可；下载文件的方式建议定期重新下载

## 📺 频道列表网页

<https://shocklele.github.io/Collect-IPTV/>

网页可按格式、分组筛选并搜索频道，同时展示台标。该页面由 `Deploy static content to Pages` 工作流部署，需要先在仓库 **Settings → Pages** 中把 **Source** 设为 **GitHub Actions** 才能访问。

---

## ⚙️ 工作原理

1. **收集**：从多个公开的 M3U / TXT 节目源汇总频道与地址（源列表见下方[节目源](#-节目源)）
2. **检测**：并发请求每条地址，过滤不可用的源并记录延迟
3. **去重优选**：同一频道只保留延迟最低的一条地址（同等条件下优先 HTTPS）
4. **分组**：按 央视频道 → 卫视频道 → 各省频道 → 主题频道（港澳台、文旅、新闻、体育、影视、少儿动漫、纪录人文、音乐、戏曲综艺）→ 其他频道 归类排序
5. **台标**：台标取自 `https://epg.112114.xyz/logo/`，生成地址前会先规范化频道名（如 `CCTV-4 中文国际` → `CCTV4`）以提高命中率；图床未收录的频道不显示台标
6. **发布**：写入 `best_sorted.m3u` / `best_sorted.m3u8`，更新本页的更新时间并自动提交

## 📡 节目源

当前使用的上游节目源（定义在 `.github/workflows/iptv.py` 末尾的 `file_urls`，按实测有效流数量从多到少排列）：

| 顺序 | 上游 | 地址 |
|---|---|---|
| 1 | [vbskycn/iptv](https://github.com/vbskycn/iptv) | `https://gh-proxy.com/raw.githubusercontent.com/vbskycn/iptv/refs/heads/master/tv/iptv4.m3u` |
| 2 | [Guovin/iptv-api](https://github.com/Guovin/iptv-api) | `https://raw.githubusercontent.com/Guovin/iptv-api/gd/output/ipv4/result.m3u` |
| 3 | [suxuang/myIPTV](https://github.com/suxuang/myIPTV) | `https://raw.githubusercontent.com/suxuang/myIPTV/refs/heads/main/ipv4.m3u` |
| 4 | [iptv-org/iptv](https://github.com/iptv-org/iptv)（中文频道） | `https://iptv-org.github.io/iptv/languages/zho.m3u` |
| 5 | [Kimentanm/aptv](https://github.com/Kimentanm/aptv) | `https://raw.githubusercontent.com/Kimentanm/aptv/master/m3u/iptv.m3u` |
| 6 | [zwc456baby/iptv_alive](https://github.com/zwc456baby/iptv_alive) | `https://raw.githubusercontent.com/zwc456baby/iptv_alive/refs/heads/master/live.m3u` |
| 7 | [hujingguang/ChinaIPTV](https://github.com/hujingguang/ChinaIPTV) | `https://raw.githubusercontent.com/hujingguang/ChinaIPTV/main/cnTV_AutoUpdate.m3u8` |
| 8 | tv.iill.top | `https://tv.iill.top/m3u/Gather` |

- 排列顺序只为便于维护，不影响结果：同一频道始终按延迟选出最优地址，与源的先后无关。
- 脚本只采集 `http://` / `https://` 地址，纯 `rtp://` 组播源或仅运营商内网可用的源不会产出任何频道，无需添加。
- 下载失败或返回非 200 的源会被跳过，不影响其余源；可在 Actions 日志中查看每个源的 `Source ...: 有效数/候选数`，长期为 0 的源建议移除。

## 📁 目录结构

| 路径 | 说明 |
|---|---|
| `best_sorted.m3u` / `best_sorted.m3u8` | 自动生成的播放列表 |
| `.github/workflows/iptv.yml` | 定时任务（每 4 小时），也可手动触发 |
| `.github/workflows/iptv.py` | 收集、检测、分组与生成脚本 |
| `.github/workflows/IPTV/` | 央视及各省频道名单，用于分组 |
| `.github/workflows/index.html` | 频道列表网页 |
| `.github/workflows/static.yml` | 将网页部署到 GitHub Pages |

## 🛠️ 自行部署与调整

- **手动更新**：仓库 **Actions → IPTV Daily Update → Run workflow**
- **fork 后首次使用**：先在 Actions 页面启用工作流；如自动提交时报 403，到 **Settings → Actions → General → Workflow permissions** 选择 **Read and write permissions**
- **更换台标图床**：修改 `iptv.py` 中的 `LOGO_URL_TEMPLATE`
- **增减节目源**：修改 `iptv.py` 末尾的 `file_urls`，并同步更新上方[节目源](#-节目源)表格
- **本地运行**：在仓库根目录执行

  ```bash
  pip install aiohttp
  python .github/workflows/iptv.py
  ```

---

## 📢 免责声明（个人学习测试专用）

本项目仅用于**网络协议、爬虫技术、自动化脚本开发等个人学习与测试用途**，不用于任何商业、盈利及违规用途。

- 所有节目源均来自互联网公开可访问链接，项目本身不生产、不存储、不篡改任何媒体内容。
- **本项目仅对收集到的公开链接进行整理、去重、可用性检测及延迟筛选，不对上游链接所提供的内容负责。**
- 部分上游节目源可能包含广告、弹窗、二维码、赞助信息或其他推广内容，**这些内容由对应的上游源自行提供，与本项目及项目作者本人无关。**
- **本项目及项目作者不代表、不授权、也不参与任何上游节目源所显示的赞助、捐赠、广告或其他商业活动。**
- 如上游页面或播放源出现要求扫码赞助、捐赠、付费等提示，请使用者自行判断，**项目作者不会以任何方式要求用户进行上述操作。**
- 严禁将本项目及生成的播放列表用于商业传播、二次分发、公开分享等行为。
- 所有频道版权均归原版权方所有，使用前请确保符合当地法律法规。
- 因违规使用本项目产生的任何法律责任、版权纠纷，均由使用者自行承担。

详细免责声明请参阅 [`DISCLAIMER.md`](./DISCLAIMER.md)。

## 🙏 致谢与许可

- 原项目：[zilong7728/Collect-IPTV](https://github.com/zilong7728/Collect-IPTV)
- 节目源：见上方[节目源](#-节目源)列出的各上游项目
- 台标来源：`epg.112114.xyz`
- 本项目基于 [Apache License 2.0](./LICENSE) 开源
