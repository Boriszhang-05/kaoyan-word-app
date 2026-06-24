# 考研单词助手 PWA

## 项目概述
全功能 PWA 考研英语单词学习应用，支持艾宾浩斯遗忘曲线复习。

- **线上地址**: https://boriszhang-05.github.io/kaoyan-word-app/
- **仓库**: https://github.com/Boriszhang-05/kaoyan-word-app (master 分支)
- **部署方式**: GitHub Pages，从 master 分支自动构建

## 文件结构
```
D:\单词\
├── index.html      # 主应用（单文件，内联 CSS/JS）
├── dict.js         # 超级词库 14,728 词（1.0 MB）
├── manifest.json   # PWA 清单
├── sw.js           # Service Worker 离线缓存
├── icon-192.png / icon-512.png             # PWA 图标
├── icon-192-maskable.png / icon-512-maskable.png  # Android 自适应图标
└── CLAUDE.md       # 本文件
```

## 技术要点

### 词库 (dict.js)
- 来源：CET4 + CET6 + 考研 + 托福 + SAT + 红宝书
- 共 14,728 词，格式：`const DICT = { "word":{zh:"释义", p:"音标", pos:"词性", ex:"例句"}, ... }`
- 查找逻辑 `lookupDict()`: 精确匹配 → 空格↔连字符互转 → 去撇号

### 导入解析 (parseImportLines)
- 支持 `单词 释义` 格式（中文字符检测边界）
- 多词短语（如 `for sure 肯定的`）：完整英文部分匹配词库 → 匹配用词典，否则用自定义释义
- 单单词：直接查词库，匹配用词典释义，否则用自定义释义

### 艾宾浩斯遗忘曲线
```js
const EB_INTERVALS = [1, 2, 4, 7, 15, 30]; // 天
// stage 0=未学, 1-6=复习中, 7=已掌握
// 复习正确 stage+1, 模糊保持, 忘记 stage-2
```

### Service Worker (sw.js)
- **每次部署前必须 bump CACHE 版本号**（当前 v9），否则用户收到旧缓存
- 用户端需刷新两次：第一次更新 SW，第二次加载新缓存
- 手动清除：chrome://serviceworker-internals/ → Unregister

### 数据存储
- localStorage，key 前缀：`wordapp_`（words/settings）
- 不同设备/浏览器数据天然隔离
- 测验选项过滤：`w.zh && w.zh.trim()` 跳过空释义词

### PWA 配置
- `display: standalone`，scope: `/kaoyan-word-app/`
- 平台感知安装引导：iOS（添加到主屏幕）/ Android（beforeinstallprompt）
- maskable 图标：紫色背景 + 安全区留白

## 部署流程
1. 修改代码
2. 更新 `sw.js` 中 `CACHE` 版本号（v9 → v10 → v11...）
3. `git add . && git commit -m "..." && git push origin master`
4. `gh api repos/boriszhang-05/kaoyan-word-app/pages/builds --method POST` 触发构建
5. 等待 1-3 分钟，检查 `curl -s https://.../sw.js | head -1` 确认版本

## 页面结构
| 页面 | 路由 | 功能 |
|------|------|------|
| 首页 | `home` | 统计 + 学习/复习入口 |
| 学习 | `learn` | 取未学词 → 闪卡 → 选择题测验 |
| 复习 | `review` | 艾宾浩斯到期词 → 看英文回忆 → 自评 |
| 词库 | `wordlist` | 全部单词列表 + 搜索 + 删除 |
| 导入 | `import` | 批量粘贴 + 词组识别 + 自定义释义 |
| 设置 | `settings` | 学习/复习数量、统计、数据管理 |

## Python 脚本注意事项
- Windows 下避免 print 中文到 stdout（GBK 编码报错 exit code 49）
- 改为写文件输出结果
- 使用 Node.js 脚本处理中文文本更稳定
