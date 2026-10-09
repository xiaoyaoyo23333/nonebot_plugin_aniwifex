# nonebot-plugin-animewifex

> 二次元老婆 · NoneBot2 插件套件 —— 完整版 + 精简版(Lite)
>
> 由 AstrBot 插件 [astrbot_plugin_animewifex](https://github.com/monbed/astrbot_plugin_animewifex) 移植而来。

群聊里的「二次元老婆」玩法：每天抽一位二次元老婆，支持**牛老婆 / 换老婆 / 交换老婆**等互动。
本仓库提供两个功能不同的版本，按群风格挑一个装。

---

## 两个版本

| | **完整版** | **Lite 版** |
| --- | --- | --- |
| 包名 | `nonebot_plugin_animewifex` | `nonebot_plugin_animewifex_lite` |
| 配置前缀 | `ANIMEWIFEX_` | `ANIMEWIFEX_LITE_` |
| 默认数据目录 | `data/animewifex` | `data/animewifex_lite` |
| 指令数 | **13** | **4** |
| 抽老婆 / 查老婆 / 换老婆 | ✅ | ✅ |
| 牛老婆 / 重置牛 / NTR 开关 | ✅ | ❌ |
| 重置换（失败禁言） | ✅ | ❌ |
| 交换老婆（发起/同意/拒绝/查看） | ✅ | ❌ |
| 管理员命令 | ✅ | ❌ |

**怎么选**

- 想要热闹的全套玩法 → **完整版**
- 只想要「每天抽一个 + 抽腻了换一个」，不想有人被抢老婆/被禁言 → **Lite 版**

两者配置前缀与数据目录完全隔离，可以同时装在一个 NoneBot2 实例里；但**指令名相同**，
默认会各答一遍，同一实例请**只启用其中一个**（或给其中一个设 `*_BLOCK_OTHER_PLUGINS=true` 独占）。

---

## 快速开始

### 安装

```bash
# 完整版
nb-cli plugin install ./nonebot_plugin_animewifex

# 或 Lite 版
nb-cli plugin install ./nonebot_plugin_animewifex_lite
```

也可以直接把插件目录放进 `src/plugins/` 下启动 NoneBot2。

> 目录名必须以 `nonebot_plugin_` 开头，`nb-cli` 才能管理它。

### 准备图片（本地图库）

两个版本都是**纯本地模式**，不联网取图。请把图片放进数据目录：

```text
data/animewifex/img/wife/        ← 完整版
data/animewifex_lite/img/wife/   ← Lite 版
├── 轻小说!明日香.jpg
├── 英雄联盟!阿卡丽.jpg
└── 明日香.jpg
```

文件名按 `作品名!角色名.jpg` 命名，机器人会据此播报「来自《作品名》的角色名」；
没有 `!` 就只报角色名。图库为空时会提示获取失败并记一条 warning。

### 配置

在 `.env` 里按前缀配置，例如完整版：

```dotenv
ANIMEWIFEX_ADMINS=["123456789"]
ANIMEWIFEX_CHANGE_MAX_PER_DAY=3
ANIMEWIFEX_TIMEZONE=Asia/Shanghai
ANIMEWIFEX_DATA_DIR=D:/nonebot-bot/data/animewifex
```

Lite 版把前缀换成 `ANIMEWIFEX_LITE_`。完整配置项见各自 README。

> ⚠️ Windows 上建议 `pip install tzdata`（判定「今天」需要 IANA 时区库）。
> 不装也能跑，会自动退化为固定 UTC+8。

---

## 指令一览（完整版）

```
【基础命令】
• 抽老婆            每天一次，随机抽一张二次元老婆
• 查老婆 @用户      查看别人的老婆（也支持昵称匹配）

【牛老婆功能】(概率较低😭)
• 牛老婆 @用户      有概率抢走别人的老婆
• 重置牛 @用户      重置牛的次数，失败会被禁言

【换老婆功能】
• 换老婆            丢弃当前老婆换新的
• 重置换 @用户      重置换老婆的次数，失败会被禁言

【交换功能】
• 交换老婆 @用户    向别人发起老婆交换请求
• 同意交换 @发起者  同意交换请求
• 拒绝交换 @发起者  拒绝交换请求
• 查看交换请求      查看当前的交换请求

【管理员命令】
• 切换ntr开关状态   开启/关闭牛老婆功能
```

Lite 版只保留 `老婆帮助` / `抽老婆` / `查老婆` / `换老婆` 四条。

### 指令是严格匹配的

只接受「命令」本身，或「命令 + 空格 + 参数」：

| 消息 | 响应 |
| --- | --- |
| `换老婆` / `查老婆 明日香` / `牛老婆 @某人` | ✅ |
| `帮我换老婆` / `换老婆啊` / `xxxx换老婆xxxx` / `换老婆。` | ❌ |

聊天里随口提到「换老婆」不会误触发。

### 共存与优先级

市面上其它「老婆」插件（`nonebot_plugin_today_waifu` 等）也注册了 `抽老婆` / `换老婆`，
而 `on_message` 默认 `block=True`——先匹配到的插件会掐断事件，后面的插件**静默无响应**。

本插件默认 `priority=0` + `block=False`：先答自己的一次元老婆，再放行给别的插件，
两者可以共存（抽群友的 + 抽二次元的各答各的）。想独占就设 `*_BLOCK_OTHER_PLUGINS=true`。

---

## 特性

- **每日重置**：次数记录、交换请求跨天自动失效，零点后定时清理
- **并发安全**：每群一把锁，连刷指令突破不了每日上限
- **数据可靠**：JSON 原子写入（临时文件 + 替换）；文件被改坏时降级加载并记日志，不会拖垮插件
- **零网络依赖**：纯本地图库，不请求外部图床
- **跨版本兼容**：nonebot2 2.2 ~ 2.5 通用（详见各 README 的「兼容性」章节）
- **不抢别人消息**：严格指令匹配 + `block=False`，与其它插件互不干扰

## 兼容环境

| NoneBot2 | OneBot 适配器 | 结果 |
| --- | --- | --- |
| 2.4.2 | 2.4.6 / 2.4.3 | ✅ |
| 2.5.0 | 2.4.6 | ✅ |

Windows 11 23H2 + Python 3.11.9 实测通过；源码按 Python 3.9+ 语法编写。

## 目录结构

```text
.
├── nonebot_plugin_animewifex/        # 完整版插件
│   ├── __init__.py                   #   主逻辑 + 消息分发
│   ├── config.py                     #   配置（pydantic）
│   ├── storage.py                    #   JSON 持久化与校验
│   ├── metadata.json                 #   NoneBot2 插件元数据
│   └── README.md                     #   详细文档
├── nonebot_plugin_animewifex_lite/   # Lite 版插件
│   └── ...
├── smoke_test.py                     # 完整版功能测试
└── smoke_test_lite.py                # Lite 版功能测试
```

## 致谢

- 上游 AstrBot 插件：[astrbot_plugin_animewifex](https://github.com/monbed/astrbot_plugin_animewifex)
- 原版思路：[astrbot_plugin_AW](https://github.com/zgojin/astrbot_plugin_AW)

## License

MIT