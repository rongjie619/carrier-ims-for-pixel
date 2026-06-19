# Carrier IMS UI Support Commerce Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 重构 Carrier IMS 主界面为 5 个底栏页面，并接入附加网络工具、广告配置、商务合作入口、配置备份恢复和 APN 自动修改入口。打赏支付只打开 DoDoPay 已有公开支持页；未配置公开页时保持未开放状态。

**Architecture:** 保留现有 `MainActivity -> MainViewModel -> ShizukuProvider -> privileged instrumentation` 主链路。App 读取 Muggle Leads 公开广告接口，商务合作提交到 Muggle Leads 合作意向接口。打赏留言和付款记录归 DoDoPay 通用能力，App 只打开 DoDoPay 公开支持页并传递金额、昵称和留言。DoDoPay 不允许新增 TurboIMS 专用表、专用接口或专用后台；TurboIMS 也不新建独立支持服务。

**Tech Stack:** Android Kotlin、Jetpack Compose Material 3、Shizuku instrumentation、SharedPreferences、HttpURLConnection、CarrierConfig/APN Provider。

---

## 文件结构

- `docs/superpowers/specs/2026-06-16-ui-support-commerce-redesign.md`：产品设计与边界。
- `docs/superpowers/plans/2026-06-16-ui-support-commerce-redesign.md`：实施计划。
- `app/src/main/java/io/github/vvb2060/ims/model/SupportModels.kt`：广告、网络出口、配置备份、APN 模型。
- `app/src/main/java/io/github/vvb2060/ims/privileged/ApnModifier.kt`：通过 Shizuku 写入 APN Provider。
- `app/src/main/java/io/github/vvb2060/ims/ShizukuProvider.kt`：新增 APN 写入桥接方法。
- `app/src/main/java/io/github/vvb2060/ims/viewmodel/MainViewModel.kt`：新增网络出口检测、Muggle Leads 广告与合作意向、DoDoPay 公开支持页配置、广告频控、配置备份恢复、APN 写入方法。
- `app/src/main/java/io/github/vvb2060/ims/ui/MainActivity.kt`：拆分 5 个底栏页面，接入新功能与区域兼容状态。
- `app/src/main/AndroidManifest.xml`：注册 APN instrumentation。
- `app/src/main/res/values/strings.xml` 和 `app/src/main/res/values-zh-rCN/strings.xml`：新增文案。

## 任务

### Task 1: 文档先行

- [x] 写设计文档，明确目标、非目标、页面职责、接口边界、验证方式。
- [x] 写实施计划，列出文件责任和任务顺序。
- [x] 纠正系统边界：商务合作和广告归 Muggle Leads；打赏留言和付款记录归 DoDoPay 通用能力；TurboIMS 不新建独立支持服务。

### Task 2: 新增模型与 APN 特权写入

- [x] 新增 `SupportModels.kt`，定义广告、IP 检测、配置备份、APN 模型。
- [x] 新增 `ApnModifier.kt`，读取 Bundle 参数并通过 `Telephony.Carriers.CONTENT_URI` 写入 APN。
- [x] 在 Manifest 注册 `ApnModifier` instrumentation。
- [x] 在 `ShizukuProvider` 增加 `applyApnConfig`。

### Task 3: 扩展 ViewModel

- [x] 增加广告 baseUrl、商务合作 baseUrl、DoDoPay 公开支持页模板和未配置判断。
- [x] 增加网络出口检测，用户触发后请求检测接口并返回结构化结果。
- [x] 增加 Muggle Leads 广告配置读取和本地弹窗频控。
- [x] 增加 Muggle Leads 合作意向提交，字段包括称呼、联系方式、合作类型和合作需求。
- [x] 打赏支付只打开 DoDoPay 公开支持页；未配置公开页时保持禁用或明确提示。
- [x] 不接入 `/api/turboims/support-*`，不创建 TurboIMS 支持订单，不读取 TurboIMS 支持记录。
- [x] 增加本地配置备份、恢复、删除。
- [x] 增加 APN 写入方法。

### Task 4: 重构主界面信息结构

- [x] 新增底栏状态和 5 个 Tab：IMS、附加、支持、合作、关于。
- [x] IMS 页保留 Shizuku、SIM、IMS 注册和核心开关；品牌信息迁移到关于页。
- [x] 从 IMS 功能列表移除 `TIKTOK_NETWORK_FIX`，迁移到附加页。
- [x] 附加页接入 Wi-Fi 修复、区域兼容状态、抖音修复、网络出口检测、APN/SIM 信息、自动 APN、快捷开关、配置备份恢复。
- [x] 支持页接入留言支持作者、DoDoPay 打赏入口和 DoDoPay 记录归属说明。
- [x] 合作页接入商务合作表单和合作广告。
- [x] 商务合作表单提交到 `https://leads.3jiezhiwai.com/api/sources/carrier-ims/intents`，不携带 API Key。
- [x] 打赏支付入口只指向 DoDoPay 公开支持页；DoDoPay 通过通用公开页创建订单。
- [x] 关于页迁移更新、仓库、Issue、日志、Dump、系统信息。
- [x] 首页广告弹窗读取配置并按本地频控展示。

### Task 5: 字符串与体验收敛

- [x] 新增中英文字符串。
- [x] 保留原有核心文案，新增说明保持短句。
- [x] 确保按钮处理中、未配置、失败、空状态都有文案。

### Task 6: 验证与修复

- [x] 运行 `./gradlew :app:assembleDebug`。
- [x] 本机实机验证使用 `-Pturboims.debugApplicationIdSuffix=.codextest` 生成共存测试包，避免覆盖已安装正式包。
- [x] 修复编译错误和资源缺失。
- [x] 检查 UI 结构是否满足设计：IMS 页不再混入附加、支持、合作和关于入口。

### Task 7: 后续 DoDoPay 通用能力，不在 TurboIMS 内实现

- [x] 如需 App 内展示打赏记录，由 DoDoPay 提供通用公开只读能力，字段保持通用，不出现 TurboIMS 专用表或接口。
- [x] DoDoPay 已上线通用付款留言能力，TurboIMS 只消费公开能力，不修改 DoDoPay。
- [x] 除非用户明确授权并先更新 DoDoPay 自身文档，否则不修改 DoDoPay 代码、数据库或线上服务。
