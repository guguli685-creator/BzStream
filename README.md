<div align="center">

# BzStream 指纹看板

**一个跑在浏览器里的"体检报告"——看看你这台设备，看起来到底像不像一个真人。**

[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](./LICENSE)
[![Single File](https://img.shields.io/badge/single--file-yes-success?style=flat-square)](./index.html)
[![Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen?style=flat-square)](./index.html)
[![Stars](https://img.shields.io/github/stars/guguli685-creator/BzStream?style=flat-square)](https://github.com/guguli685-creator/BzStream)

[在线体验](https://guguli685-creator.github.io/BzStream/) · [功能特性](#功能特性) · [检测项](#检测了什么) · [快速开始](#快速开始) · [English](#english)

</div>

---

## 这是什么

现代网站早就不靠 Cookie 认人了。显卡型号、字体渲染的微小差异、Canvas 画图的像素偏差、屏幕外框尺寸、甚至你的帧率稳不稳——这些拼起来就是一个几乎唯一的**指纹**。

而自动化工具（Puppeteer、Playwright、Selenium、各类无头浏览器）在这套体系下破绽尤其多：`navigator.webdriver` 会亮红灯、窗口外框高度是 0、显卡字符串写着 SwiftShader、帧率稳到不像话。

**BzStream 指纹看板把这套"被审查的视角"反过来交到你手里。** 打开页面，3 秒后你会拿到一份 19 项的体检报告：哪些维度正常，哪些维度正在暴露你，以及一份可以直接丢给后端的结构化 JSON。

> ⚠️ 本项目用于**自查、调试与安全研究**。前端能读到的一切都可以被伪造，请勿把评分当作唯一判据。

---

## 功能特性

- 🧩 **单文件零依赖** — 一个 `index.html` 搞定，无框架、无构建、无 CDN，下载下来双击就能跑
- 🔒 **纯本地计算** — 指纹数据不上传任何服务器，只有 WebRTC 与出口 IP 两项会向 STUN / ipify 发起一次请求
- 🌐 **中英双语** — 一键切换，切换时重新渲染而非重新采集，瞬间生效
- 🌗 **深浅主题** — 跟随系统，也可手动切换，选择本地保存
- 📱 **全端自适应** — 手机与桌面各一套布局，顶栏下拉菜单收纳全部功能
- 🛡 **容错设计** — 每项检测独立兜底，单项失败不影响其余；所有异步项带超时保护
- 📤 **结构化输出** — 一键复制完整指纹 JSON，字段对齐服务端交叉验证的需求

---

## 检测了什么

共 **19 项**，分为 5 组：

| 分组 | 检测项 |
| :--- | :--- |
| **自动化痕迹** | `navigator.webdriver` · CDP 注入变量 · 原生函数完整性 · `window.chrome` 运行时 · Permissions 一致性 |
| **窗口与屏幕** | 外层窗口尺寸 / 屏幕坐标 · 屏幕 / 视口 / DPR / 色深 |
| **渲染指纹** | WebGL 渲染器 · WebGL 扩展集与着色器精度 · Canvas 渲染哈希 · Audio 音频指纹 |
| **设备与网络** | CPU 核心 / 内存 · 触摸能力与指针 · 媒体设备枚举 · 电池状态 · **WebRTC 泄漏检测** |
| **时间与性能** | rAF 帧率与抖动 · 时钟 / 计时器一致性 · 语言 / 时区 |

<details>
<summary><b>几个比较"狠"的检测项展开</b></summary>

- **原生函数完整性** — 用 `Function.prototype.toString` 校验 `querySelector`、`toDataURL`、`getParameter` 等 9 个关键 API 是否仍是 `[native code]`。这是抓"指纹伪装插件"最有效的一招。
- **外层窗口尺寸** — 无头 / 虚拟显示器下 `outerWidth / outerHeight` 常为 0，或与视口完全相等，这是很难伪装的破绽。
- **WebRTC 泄漏** — 收集 ICE 候选地址并与 HTTP 出口 IP 比对。**两者不一致即说明真实 IP 正在绕过代理泄漏**，自建节点用户务必关注。
- **Canvas 纯色检测** — 不只算哈希，还会采样像素判断是否被反指纹手段干扰成了纯色块。
- **rAF 帧率稳定性** — 采样 1.2 秒内的帧间隔。帧率无上限或抖动小于 0.3ms，指向虚拟显示器。

</details>

---

## 评分怎么读

总分 **0–100**，由命中的风险项加权得出：

| 分数 | 判定 | 含义 |
| :---: | :--- | :--- |
| `0 – 24` | 🟢 低风险 | 未发现明显的自动化痕迹，这台机器"看起来像真人" |
| `25 – 59` | 🟡 中风险 | 存在若干可疑特征，建议逐项查看说明 |
| `60 – 100` | 🔴 高风险 | 多项高危特征同时命中，基本可确定是受控环境 |

> 权重写在前端仅为方便自查。生产环境请把原始观测值交给服务端做交叉验证。

---

## 快速开始

### 在线体验

直接访问：**https://guguli685-creator.github.io/BzStream/**

### 本地使用

```bash
git clone https://github.com/guguli685-creator/BzStream.git
cd BzStream
# 双击 index.html，或用任意静态服务器打开
python3 -m http.server 8080
```

> 💡 部分 API（媒体设备枚举、服务端时间比对）需要**安全上下文**，用 `https://` 或 `localhost` 打开才能完整工作。`file://` 下会显示"不适用"，属正常现象。

---

## 导出的 JSON

点击右上角「复制」，你会拿到这样一份数据：

```json
{
  "timestamp": "2026-09-20T04:12:33.128Z",
  "locale": "zh-CN",
  "score": 0,
  "verdict": "Low Risk",
  "hardware": {
    "cores": 8,
    "memory": 8
  },
  "environment": {
    "screen": "424x942 (viewport: 424x772)",
    "ua": "Mozilla/5.0 ...",
    "platform": "Linux aarch64",
    "languages": "zh-CN,zh",
    "timezone": "Asia/Shanghai",
    "devicePixelRatio": 2.625,
    "colorDepth": 24,
    "maxTouchPoints": 5,
    "chromeRuntime": true,
    "plugins": 0,
    "outer": { "width": 424, "height": 942, "screenX": 0, "screenY": 0 }
  },
  "render": {
    "webgl": "Adreno (TM) 650",
    "webglVendor": "Qualcomm",
    "webglExtensions": { "count": 34, "hash": "0x1b2251c8fc9f8", "maxTextureSize": 16384 },
    "canvasHash": "0x5a18fea0",
    "audioHash": "0x7c31d0b2a4e1"
  },
  "device": { "mediaDevices": "mic 2 / cam 1 / spk 3", "battery": "charging · 100%" },
  "network": {
    "webrtcPublicIps": [],
    "webrtcPrivateIps": ["192.168.1.5"],
    "mdnsCandidates": 2,
    "httpEgressIp": "203.0.113.7",
    "egressNote": null
  },
  "timing": {
    "rafFps": 60.1,
    "rafJitterMs": 1.24,
    "rafFrames": 72,
    "timerResolutionMs": 0.1,
    "clockSkewMs": -3,
    "serverTimeOffsetMs": 128
  },
  "automation": {
    "webdriver": false,
    "nativeIntegrity": { "checked": 9, "failed": [] }
  },
  "riskFlags": [],
  "riskDetail": []
}
```

`riskDetail` 里带权重，方便后端做二次加权或阈值调整：

```json
"riskDetail": [
  { "key": "outer", "weight": 25, "detail": "outerWidth/outerHeight = 0x0" }
]
```

---

## 项目结构

```
BzStream/
├── index.html      # 全部内容：UI + 检测逻辑 + i18n（单文件，无依赖）
├── README.md
└── LICENSE
```

纯原生 HTML / CSS / JavaScript，没有构建步骤。想改哪里直接改 `index.html` 就行。

---

## 自定义

打开 `index.html`，搜索 `PROJECT`，把仓库地址换成你自己的：

```js
var PROJECT = {
  name: 'BzStream Fingerprint Board',
  repo: 'https://github.com/guguli685-creator/BzStream',
  issues: 'https://github.com/guguli685-creator/BzStream/issues',
  license: 'MIT',
  version: '1.4.0'
};
```

想再加一门语言？在 `I18N` 对象里加一个语言键，复制 `en` 的键值结构翻译一遍即可，界面会自动出现在语言列表里。

---

## 浏览器兼容

| 浏览器 | 支持情况 |
| :--- | :--- |
| Chrome / Edge (Chromium ≥ 90) | ✅ 完整支持 |
| Safari (iOS / macOS 15+) | ✅ 支持，电池 / deviceMemory 等 Chromium 专有 API 显示"不适用" |
| Firefox | ✅ 支持，同上 |
| 各类 WebView | ⚠️ 基本可用，部分 API 取决于宿主实现 |

---

## 常见问题

<details>
<summary><b>为什么某项显示"接口不支持"？</b></summary>

部分 API 是 Chromium 专有的（`navigator.getBattery`、`navigator.deviceMemory`），Firefox / Safari 上不存在，属于正常表现，并不代表环境异常。
</details>

<details>
<summary><b>分数是 0，就代表我完全不会被识别吗？</b></summary>

不代表。本工具只覆盖客户端侧可读的维度，服务端还能看到你的 IP、ASN、TLS 指纹（JA3/JA4）、HTTP/2 指纹、Header 顺序等，这些是 JS 拿不到的。真正的风控判断需要两端结合。
</details>

<details>
<summary><b>为什么 WebRTC 那一项有时候很慢？</b></summary>

它需要向 STUN 服务器收集 ICE 候选地址，最坏情况会等到超时（2.2 秒）。如果网络环境屏蔽了 STUN，就会走超时分支并显示"未获取到候选地址"。
</details>

---

## 路线图

- [ ] 服务端上报接口：把 JSON + IP / ASN / JA3 一起回显
- [ ] 一致性交叉矩阵：UA × 时区 × 语言 × 屏幕 × 触摸的两两矛盾检测
- [ ] 历史记录对比：与上次检测结果做 diff
- [ ] 二维码分享：手机扫一扫在移动端复核同一环境

---

## 贡献

欢迎提 [Issue](https://github.com/guguli685-creator/BzStream/issues) 反馈检测误报，或补充新的检测维度。提交 PR 前请保持"单文件、零依赖"这一约束。

---

## 许可证

[MIT](./LICENSE) © 2026 BzStream

---

## 免责声明

本项目仅供学习、研究与自建服务的合规自查使用。请勿用于绕过他人网站的安全机制或从事任何违法活动。使用者需自行承担因使用本工具而产生的一切后果。

---

<div align="center">

## English

**BzStream Fingerprint Board** is a single-file, zero-dependency browser fingerprint inspector.

It runs 19 checks across five groups — automation traces, window & screen, rendering fingerprints, device & network, and timing — then produces a 0–100 risk score plus a structured JSON payload ready for server-side cross-validation.

Everything runs locally; no fingerprint data is uploaded. Just open `index.html` and wait three seconds.

MIT Licensed.

</div>
