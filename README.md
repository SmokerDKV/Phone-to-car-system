# CarMode · 手机改车机

把一台安卓手机变成车机：一个全屏的车机桌面 App，带仪表盘、实时地图、地图选点导航和网易云音乐控制条。
**保留手机原有功能**——按 Home 键随时退回系统桌面，不影响任何原有应用。

```
项目：手机改车机（CarMode）
设备：HUAWEI P50E（Android）
形态：WebView 壳 APK + 单文件 HTML 界面
版本：v1.2（versionCode 3）
兼容：Android 5.0+（minSdk 21 / targetSdk 34），任意安卓机型通用
```

---

## 一、它长什么样 / 能做什么

界面分左右两栏（横屏）或上下两段（竖屏），顶栏与侧栏都支持一键切换横竖屏。

### 仪表盘（主区）
- SVG 270° 弧形速度表，大字号数字 + 弧形进度 + 端点亮标
- 量程可切 **160 / 240 / 320** km/h，刻度随量程动态重建
- 数字颜色跟着速度走：青 → 白 → 红
- 底部四个数据格：最高 / 平均 / 里程 / 时长
- **速度感**：背景透视网格持续向前滚动（车速越快滚得越快）、22 条横向流光线条、径向辉光与数字辉光随速度增强

### 小地图（侧栏上）
- **腾讯位置服务 GL JS** 渲染的真实底图 + 实时路况图层
- 车辆箭头、真实行驶轨迹线、**车头朝上自动旋转**（指北针 N 绕边缘转）
- **车速越快视野自动拉远**（GTA 的原生行为）
- 点一下展开**全屏地图**
- 无 Key / 离线时自动退回 **GTA 风格风格化路网**（见下文「兜底」）

### 地图选点导航（全屏地图）
- 工具栏图钉按钮 → 点击地图任意位置 → 落红色图钉 → 底部卡片显示坐标与地名 → 「开始导航」
- 唤起身机上的**百度地图**开始驾车导航
- 进入选点模式会自动暂停视野跟随（否则刚点的远处位置会被立刻拉回）

### 播放器控制条（侧栏下）
- 左：专辑封面（无图时显示音符图标，播放中变主题色）
- 中：歌名 + 「网易云」来源标签 + 歌手/时长
- 右：⏮ ▶/⏸ ⏭ 三个圆形按钮，主按钮带发光
- 底边贴一条 3px 进度条
- **真的能控制网易云音乐**（见下文「原生媒体层」）

### 其它
- 时钟 / 日期、电量、GPS 状态芯片
- **屏幕常亮**（车机刚需）、沉浸式全屏
- 主题色切换（5 种）
- **返回系统桌面**按钮，保证随时能回到原系统
- 设置面板：方向 / 速度来源 / 单位 / 量程 / 主题色 / 实时地图 Key / 路况开关

---

## 二、目录结构

```
手机改车机/
├── README.md                  ← 本文件
├── carmode/                   ← 前端界面（单文件、零外部依赖）
│   ├── index.html             ← 全部界面 + 逻辑（1767 行，约 76 KB）
│   ├── manifest.json          ← PWA 清单（fullscreen / 横竖屏 any）
│   ├── sw.js                  ← Service Worker（离线缓存，仅接管同源请求）
│   ├── icon.svg               ← 矢量图标
│   ├── icon-192.png           ← 图标 192
│   └── icon-512.png           ← 图标 512
└── apk/
    ├── CarMode_v1.0.apk       ← 初版
    ├── CarMode_v1.1.apk       ← 接入实时地图 + 原生媒体层
    └── CarMode_v1.2.apk       ← 【当前】修复小地图 + 地图选点导航
```

> 安卓壳工程在 `D:\android-build\`（纯手工工具链，**不依赖 Gradle**），不在本目录内。

---

## 三、安装到手机

1. 把 `apk/CarMode_v1.2.apk` 传到手机（或直接从 `Download/` 目录找已推送的包）
2. 在手机上点击安装；首次会提示「允许此来源安装应用」，同意即可
3. 桌面出现「车机模式」图标，点开即全屏车机界面

**升级提示**：v1.1 及以后使用同一把签名证书，**可以直接覆盖安装，无需卸载**。
（只有从 v1.0 升到 v1.1 那一次因为换过密钥必须卸载重装，这个坑已修掉。）

### 首次使用建议做两件事

| 事项 | 位置 | 说明 |
|---|---|---|
| 填地图 Key | 车机 ⚙ 设置 →「实时地图」 | 不填则小地图退回风格化路网，其它功能不受影响 |
| 开「通知使用权」 | 车机设置里的「去开启」按钮 | 播放器才能显示真实歌名/歌手/封面；**不开启也能正常控制播放** |

---

## 四、地图 Key 怎么申请（腾讯位置服务）

> 合规要求：地图底图只能用 **腾讯 / 高德 / 百度 / 天地图**，且**不得内置可用的 Key**。
> 本项目所有 Key 都由用户在运行时自己填。

**为什么 APK 里必须自己填**：WorkBuddy 预览环境走的是官方密钥代理，而那是跑在**电脑本地 127.0.0.1** 上的服务，手机 WebView 根本访问不到。

步骤：

1. 打开 <https://lbs.qq.com/dev/console/key/manage>（QQ / 微信登录，无需单独注册）
2. 完成**个人实名认证**（免费，几分钟；不完成的话创建应用会卡住）
3. 创建应用（类型选 Web 端）→ 添加 Key
4. **Key 类型必须勾选「JavaScript API GL」**
5. **域名白名单填 `*`**
   - 车机页面在 APK 内是以虚拟域名 `https://carmode.local/` 加载的（为了让定位、离线缓存可用），填具体域名必然对不上；腾讯官方文档也说明本地开发可填 `*`
   - 想更严格可以试 `carmode.local`，能通就用它
6. 复制 Key（约 30 位字符，形如 `ABCDE-FGHIJ-KLMNO-PQRST-UVWXY-Z1234`）
7. 车机 ⚙ 设置 →「实时地图」→ 粘贴 → 保存
   - 若小地图已在运行（风格化兜底），会提示「重开应用后生效」，把车机后台划掉重开即可
   - 成功后小地图上会出现「实时地图」小标签

**如果腾讯这边通不过**（比如 `*` 不给过、或不想实名），可以换**百度地图**——百度的 Referer 白名单可留空不限，门槛更低，代价是 Key 类型要选「浏览器端」，前端需改两行加载代码。

---

## 五、原生媒体层：播放器怎么连上网易云

网页控制不了别的 App，必须走原生。分成两级能力：

| 能力 | 状态 | 是否需要授权 |
|---|---|---|
| 播放 / 暂停、上一首、下一首 | ✅ 可用 | **不需要任何权限** |
| 显示歌名 / 歌手 / 进度 / 封面 | ✅ 可用 | 需开一次「通知使用权」 |

- **控制这条路零权限**：优先直接操作媒体会话（`MediaController.TransportControls`）；
  拿不到会话时退化成系统媒体按键事件（`AudioManager.dispatchMediaKeyEvent`），系统会自动路由给当前活跃的播放器。
- **显示需要「通知使用权」**：这是安卓的硬规定——App 想读别的应用的播放信息，必须有一个 `NotificationListenerService` 组件被授权。
  本项目新增的 `CarNotificationListener` **只作为授权载体，不监听任何通知内容**。
- 点播放器封面可直接唤起网易云音乐。

---

## 六、技术要点与踩过的坑

这一节是给以后改代码的人的，很多是花时间才查出来的。

### WebView 壳
- 页面**不走 `file://`**，而是自建虚拟域名 `https://carmode.local/`，由 `WebViewClient.shouldInterceptRequest` 从 assets 读取返回。
  这样页面处于 **secure origin**，定位 / Service Worker 才能正常工作。
- `intent://` 由原生 `Intent.parseUri` 解析后 `startActivity`，用于唤起百度地图 / 网易云音乐。
- 注入 `window.CarNative` 桥接对象，补齐网页拿不到的能力（见下表）。
- 沉浸式全屏 + `FLAG_KEEP_SCREEN_ON`；`configChanges` 含 `orientation`，旋转不重建页面。
- Manifest 里用 `<queries>` 声明 MAIN/LAUNCHER，规避 Android 11+ 的包可见性限制。

### `window.CarNative` 完整契约

| 方法 | 说明 | 需权限 |
|---|---|---|
| `isNative()` / `battery()` / `charging()` | 环境识别与电量 | 否 |
| `setKeepAwake(bool)` / `toggleFullscreen()` | 屏幕常亮 / 沉浸式切换 | 否 |
| `mediaCmd('toggle'\|'play'\|'pause'\|'next'\|'prev'\|'stop')` | 控制活跃播放器 | **否** |
| `mediaInfo()` | JSON：playing/title/artist/position/duration/artKey | 是（通知使用权） |
| `mediaArt()` | 封面 `data:image/jpeg;base64`（144px） | 是 |
| `hasNotificationAccess()` / `openNotificationAccess()` | 查询 / 跳转授权页 | 否 |
| `openMusic()` | 启动 `com.netease.cloudmusic` | 否 |

### 地图踩坑
- **地图容器必须「先可见、再 `new TMap.Map`」**。容器 `display:none` 时尺寸为 0，地图初始化会**静默失败**（不报错，只是不显示）。
  正确顺序：给容器加显形类 → `new Map` → 延迟约 320ms 调 `map.resize()`。
  ⚠️ v1.0 小地图一直退回风格化底图就是这个原因；全屏地图当时做对了，小地图漏了这一步。
- 小地图容器设 `pointer-events:none`，让点击穿透，保证「点小地图 = 展开全屏」不被地图 SDK 吞掉。
- **坐标系**：腾讯输出 **GCJ-02**。唤起百度地图导航用
  `baidumap://map/direction?destination=latlng:LAT,LNG|name:XXX&coord_type=gcj02&mode=driving&src=...`
  **不需要 BD-09 转换**（用 `coord_type=gcj02` 声明即可）。
- 逆地理编码 `TMap.Geocoder.getAddress()` 依赖 Key 的 **WebService 权限**（不一定有），**必须全程 `try/catch`**，失败降级为只显示经纬度。
- **选点模式必须停掉视野跟随**：跟随逻辑每 0.2s `setCenter` 到车辆，不停跟随的话地图会被反复拉回，用户点不到远处位置。
- 车辆图标必须用**内联 SVG / base64**，不得引用官方 demo 图片路径。

### Service Worker 陷阱（已修）
`sw.js` 的 fetch 拦截**必须只接管同源请求**。原实现对所有 GET 做 cache-first，离线时回退成 `index.html`——等于把 HTML 当 JS 返回，会直接搞崩地图 SDK。
现在：离线回退仅限 `req.mode === 'navigate'`。

### SVG 陷阱
**SVG 元素不继承 `HTMLElement`，不支持 `hidden` 属性**。设 `svgEl.hidden = true` 只是挂了个无效属性，图标不会隐藏。
播放/暂停图标切换改成用 CSS 状态类实现。

### 构建链踩坑
1. **build-tools 34.0.0 自带的 `d8` 有 bug**：凡是「捕获外部实例的内部类 / 匿名类」（class 里有 `this$0` 字段）都会抛
   `NullPointerException: Cannot invoke "String.length()"`。
   → 改用 Google Maven 上正式发布的 **R8 9.5.26** 替代（`java -cp r8.jar com.android.tools.r8.D8 ...`）。
2. `d8` 逐文件传参在 Windows 下也易出问题 → 先用 Python 打成 `classes.jar` 再传给它。
3. `javac` 用 `-source 8 -target 8 -bootclasspath android.jar`（JDK 21 可用，class 版本 52）。
4. **构建目录必须放纯 ASCII 路径**（`D:\android-build`），项目目录含中文。
5. **签名证书必须持久化**：脚本开头的 `rm -rf out/` 会连 keystore 一起删，导致每次构建换一把密钥、已装版本永远无法覆盖升级。
   现在证书固定在 `D:\android-build\keys\carmode.keystore`。

---

## 七、从源码重新构建 APK

环境（本机已就位）：

| 组件 | 路径 |
|---|---|
| JDK | `C:\Program Files\Microsoft\jdk-21.0.10.7-hotspot` |
| Android SDK | `D:\android-build\sdk`（cmdline-tools + `platforms;android-34` + `build-tools;34.0.0`） |
| R8（替代 d8） | `D:\android-build\r8.jar` |
| 签名证书 | `D:\android-build\keys\carmode.keystore`（口令 `carmode123`，alias `carmode`） |
| 壳工程 | `D:\android-build\app\`（AndroidManifest.xml / java/ / res/ / assets/） |

构建流程：

```bash
# 1. 同步前端资源到壳工程 assets
cd /d/android-build
cp "D:/workbuddy项目目录/手机改车机/carmode/"* app/assets/

# 2. 修改 app/AndroidManifest.xml 里的 versionCode / versionName（升级打包时）

# 3. 一把梭构建
bash build.sh
# 产出：D:/android-build/out/CarMode.apk

# 4. 归档
cp out/CarMode.apk "D:/workbuddy项目目录/手机改车机/apk/CarMode_v1.x.apk"
```

`build.sh` 内部步骤：aapt2 compile → aapt2 link（打包 assets）→ javac → 打包 classes.jar → R8/D8 生成 dex → 写入 APK → zipalign → 生成/复用签名证书 → apksigner 签名 → 校验。

---

## 八、把文件传到手机（MTP 通道）

本机没有 adb，走 Windows Shell COM（`Shell.Application`）。这是已验证可用的方法：

```powershell
$sh = New-Object -ComObject Shell.Application
$pc = $sh.NameSpace(17)                                              # 此电脑
$phone = $pc.Items() | ? { $_.Name -eq "HUAWEI P50E" }               # 精确匹配，别用模糊匹配
$sf = ($phone.GetFolder.Items() | ? { $_.Name -eq "内部存储" }).GetFolder
```

硬性要求：

1. **必须放开沙箱执行**（沙箱默认禁止 COM 实例化）
2. **`.ps1` 脚本必须存为 UTF-8 with BOM**，否则 PowerShell 5.1 按 GBK 解析，中文目录名匹配必失败
3. 枚举 COM 集合统一写成 `@($folder.Items())`

能力边界：

- ✅ 读 / 列目录、拷出文件、写入新文件、重命名、移动进出
- ❌ **不能原地覆盖已存在文件**、**不能删除**

**替换手机上已有文件的做法**：先把旧文件移出到电脑，再把新文件写进去。

> ⚠️ 本机没有 adb。装一个 adb 会让后续调试顺畅得多（可覆盖、可删除、可 adb install）。

---

## 九、已知限制 / 路线图

### 限制
- **车速依赖 GPS**，在地库、隧道里会丢信号。更准的做法是接 **OBD 蓝牙适配器**读车辆总线——原生层已经打通，后续接 OBD 是顺理成章的事。
- **风格化路网是示意，不是真实道路**（合规原因无法取用地图数据）；但**行驶轨迹是真实的**（GPS 经纬度按等距圆柱投影换算成米）。
- **选点导航依赖手机上装有百度地图**。
- 没填腾讯 Key 时，小地图退回风格化路网。

### 可能的方向
- 接 OBD 读真实车速 / 转速 / 油耗
- 地图底图可切换（百度 / 高德）
- 播放器追加音量控制、循环 / 随机
- 后台保活、开机自启

---

## 十、版本历史

| 版本 | 日期 | 内容 |
|---|---|---|
| v1.0 | 2026-10-14 | 首个 APK：仪表盘 + 速度动效 + 应用宫格 + 横竖屏 |
| v1.1 | 2026-10-14 | 接入腾讯实时地图（含路况）+ 原生媒体层（控制网易云）+ 修 `sw.js` / 签名证书两个 bug |
| v1.2 | 2026-10-14 | **修复小地图不显示真实底图**（容器可见性顺序）+ 新增**地图选点导航** |

---

## 十一、相关约定（给后续协作者）

- 对手机的任何写操作**先确认**；测试完必须还原（移出文件并核对目录条目数）。
- 地图只用腾讯 / 高德 / 百度 / 天地图；**不得内置任何可用的地图 Key**。
- 腾讯地图 AppKey 相关能力若要打包，注意 Key 只能运行时由用户填写。
