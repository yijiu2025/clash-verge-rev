# Clash Verge Rev — 个人构建（含节点测速）

> 基于上游 [clash-verge-rev/clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) **v2.5.9** 的个人 fork。
> 只做了一件事：**在延迟测试之外，再给每个节点测一次下载速度（峰值带宽）**。
>
> 上游 PR 已关闭，本项目仅在个人仓库维护，不追求合入上游。

- 当前版本：**2.5.10**
- 平台：**Windows x64**（Linux / macOS 未验证）
- 安装包下载：[Releases](https://github.com/yijiu2025/clash-verge-rev/releases)

---

## 与上游的差异

| 能力 | 上游 v2.5.9 | 本 fork |
| --- | --- | --- |
| 节点延迟测试 | 有 | 有 |
| **节点测速（峰值带宽）** | 无 | **有**，组工具栏一键测全组 |
| **自定义测速地址** | 无 | **有**，组工具栏输入框常驻 |
| **网页地址测速** | 无 | **有**，自动抓页面资源并发下载 |
| **多节点并发测速** | 无 | **有**，3 路并发 |
| 暂停 / 继续 / 取消 | 无 | **有** |
| 其余功能 | — | 与上游完全一致 |

改动范围：`src-tauri/src/feat/clash.rs`、`src/components/proxy/*`、`src/services/speed.ts`、语言包。其余代码未动。

---

## 下载与安装

### 全新安装（推荐）

下载 **`Clash.Verge_2.5.10_x64-setup.exe`**，右键「以管理员身份运行」→ 选择安装路径 → 下一步到底。

安装包自带：

- 主程序
- mihomo 内核（`verge-mihomo` 与 `verge-mihomo-alpha`）
- WebView2 运行库引导程序（老系统自动安装）
- 系统服务（TUN 模式依赖）
- 开始菜单快捷方式 + 卸载程序

系统要求：**Windows 10 / 11，64 位**。

### 覆盖升级（已装过）

下载 **`Clash.Verge_2.5.10_x64_portable.exe`**，关闭正在运行的 Clash Verge，用它覆盖安装目录里的 `clash-verge.exe`，重新启动即可。配置、订阅、规则都不会丢。

> 便携版 exe 不带内核与系统服务，**不能**在全新电脑上单独使用。

---

## 测速怎么用

### 基本操作

1. 打开「代理」页
2. 每个代理组标题那一行就是工具栏，从左到右：
   - 定位当前节点、**测延迟**、**测速**、排序、显示详情、搜索
3. 工具栏上的输入框就是**测速地址**，可以留空
4. 点 **测速按钮**（播放图标）开始

### 测试中的交互

| 操作 | 效果 |
| --- | --- |
| 点一次 | 开始测速，3 个节点同时进行，按钮变成转圈 |
| 转圈时再点 | **暂停**（当前节点测完就停，不发新任务） |
| 暂停后点播放图标 | **继续** |
| 在按钮上**右键** | **取消**整轮测速 |
| 鼠标悬停按钮 | 显示「当前测到哪个节点 / 已完成几个 / 总进度 / 正在使用的测速地址」 |

每个节点测 **8 秒**（前 2 秒是 TCP 慢启动和 TLS 握手，属于预热不计入峰值）。整组测完的时间约为「节点数 ÷ 3 × 8 秒」。

每次开始新一轮会先清掉上一次的错误提示，避免旧报错干扰判断。

### 测速地址怎么填

| 你填的内容 | 实际行为 |
| --- | --- |
| 留空 | 用「设置 → 测速地址」里选的源（默认 OVH 100MB） |
| `https://proof.ovh.net/files/100Mb.dat` | 直接下载该文件测带宽（**最准**） |
| `www.google.com`（不带协议） | 自动补成 `https://www.google.com`，按网页模式测 |
| 随手写的文字（如「亏本机场」） | 判定为非地址，自动回落到默认源，不会导致全组失败 |

### 两种模式

**文件直链模式**（推荐）

填一个返回大文件的直链，程序开 4 条连接同时下载、统计峰值速率。这是最准确的测法，数字可直接当作节点带宽。

**网页模式**

填网站首页（比如 `www.baidu.com`、`www.google.com`）时：

1. 先探测页面（预算 10 秒，自动跟随最多 5 次重定向）
2. 从 HTML 里提取图片 / CSS / JS 等资源，**同域名优先**
3. 把这些资源排成队列，4 路并发下载来估算带宽

请求带完整浏览器头（Chrome UA / Accept / Accept-Language），避免被 CDN 的反爬机制拒绝。

因为网页资源通常不大、单个资源下载完就结束，**网页模式测出的数值普遍低于文件直链模式**，只适合「粗略比较节点快慢」，不适合看绝对带宽。

### 内置测速源

| 源 | 说明 |
| --- | --- |
| OVH（100 MB） | 默认。实测最稳定，国际线路首选 |
| Cloudflare（50 MB） | 速度快的线路可用，但会拒绝过大的 `bytes` 参数，偶尔 TLS 握手失败 |
| CacheFly（100 MB） | 备用国际源 |
| TUNA 清华镜像（ISO） | 测国内直连带宽，走代理测反而不准 |

---

## 常见问题

**Q：为什么整组一下全失败？**

先看悬停提示里的具体错误。常见原因：测速地址填了非法内容（会自动回落，但填成半截网址也可能失败）、当前节点线路本身不通、或测速源在你线路上被阻断。**换一个测速源重试**是最快的排查手段。

**Q：为什么 www.google.com 也能测？它不是网页吗？**

会自动识别成网页并走网页模式。`google.com` 会 302 两次跳到 `google.com.hk`，程序会自动跟随重定向（最多 5 次）后再抓资源。

**Q：为什么测出来的速度比实际带宽低？**

- 3 个节点并发测，共享你家的总带宽，理论上每个只剩 1/3
- 网页模式本身偏低估
- 测速源服务器出口带宽有限

**节点之间的快慢排序是准的**，绝对数值当作参考即可。想要准确数值，用文件直链源，并且等别的设备空闲时测。

**Q：为什么显示「该地址返回的是网页而不是可下载的文件」？**

这句话只在一种情况出现：你填的文件直链实际返回了 HTML（地址填错了，或者该链接需要登录/被重定向到错误页）。换成真正的文件地址即可。

**Q：测一次会消耗多少流量？**

8 秒 × 4 条连接。百兆线路一个节点约 100 MB，一组 20 个节点最坏情况约 2 GB。流量敏感的场景建议只在少数节点上测。

**Q：测速结果会保留吗？**

保留 30 分钟。重开应用或超过 30 分钟后重新测试。

---

## 从源码构建

前置要求：Rust 工具链、Node 20+、pnpm、Visual Studio Build Tools（MSVC + Windows SDK）。

```shell
pnpm i
pnpm run prebuild

# 完整安装包（NSIS）
pnpm exec tauri build --config tauri.local.conf.json

# 只要可执行文件，不打包
pnpm exec tauri build --config tauri.local.conf.json --no-bundle

# 开发模式（前端热重载）
pnpm exec tauri dev
```

- `tauri.local.conf.json`：关闭更新检查与 updater 产物，避免个人构建被上游更新覆盖
- 产物位置：`target/release/clash-verge.exe`、`target/release/bundle/nsis/*-setup.exe`
- 开发模式若窗口白屏，多半是 Vite 只监听了 IPv6：`vite.config.mts` 已固定 `host: '127.0.0.1'`，`tauri.conf.json` 的 `devUrl` 也保持一致

### 主要改动文件

| 文件 | 作用 |
| --- | --- |
| `src-tauri/src/feat/clash.rs` | 测速后端：建连、浏览器头、重定向跟随、网页资源探测、4 路并发下载、并发选节点锁 |
| `src/services/speed.ts` | 测速地址校验与回落、内置测速源、错误信息提取、缓存 |
| `src/components/proxy/proxy-groups.tsx` | 组测速调度：3 条并发流水线、暂停/继续/取消、进度状态 |
| `src/components/proxy/proxy-head.tsx`、`proxy-group-tools.tsx` | 工具栏按钮与测速地址输入框 |
| `src/locales/{zh,en}/proxies.json` | 相关文案 |

---

## 上游原有功能

- 基于 Rust + Tauri 2，内置 [mihomo](https://github.com/MetaCubeX/mihomo) 内核，可切换 Alpha 内核
- 系统代理与守卫、TUN（虚拟网卡）模式
- 配置管理、Merge / Script 增强、语法提示
- 可视化节点与规则编辑、自定义主题、托盘图标、CSS 注入
- WebDAV 配置备份与同步

上游文档与常见问题：<https://clash-verge-rev.github.io/>

---

## 隐私

不收集任何用户数据，配置与日志仅保存在本地。详见 [隐私政策](./PRIVACY.md)。

## 许可

GPL-3.0，详见 [LICENSE](./LICENSE)。

## 致谢

本项目是 [clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) 的衍生构建，其上游又基于：

- [zzzgydi/clash-verge](https://github.com/zzzgydi/clash-verge)
- [tauri-apps/tauri](https://github.com/tauri-apps/tauri)
- [MetaCubeX/mihomo](https://github.com/MetaCubeX/mihomo)
- [Dreamacro/clash](https://github.com/Dreamacro/clash)
- [Fndroid/clash_for_windows_pkg](https://github.com/Fndroid/clash_for_windows_pkg)
