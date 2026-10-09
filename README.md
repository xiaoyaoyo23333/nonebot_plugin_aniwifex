# nonebot-plugin-animewifex

> 二次元老婆 · NoneBot2 插件套件 —— 完整版 + 精简版（Lite）
>
> 由 AstrBot 插件 [astrbot_plugin_animewifex](https://github.com/monbed/astrbot_plugin_animewifex) 移植而来。
>
> 开发过程中使用 Minimax M3.1-Flash-Preview 辅助。

群聊里的「二次元老婆」玩法：每天抽一位二次元老婆，支持**牛老婆 / 换老婆 / 交换老婆**等互动。
本仓库提供两个功能不同的版本，按群风格挑一个装。

---

## 两个版本怎么选

| | **完整版** | **Lite 版** |
| --- | --- | --- |
| 包名 | `nonebot_plugin_animewifex` | `nonebot_plugin_animewifex_lite` |
| 配置前缀 | `ANIMEWIFEX_` | `ANIMEWIFEX_LITE_` |
| 默认数据目录 | `data/animewifex` | `data/animewifex_lite` |
| 指令数 | **13** | **4** |
| 抽老婆 / 查老婆 / 换老婆 | ✅ | ✅ |
| 牛老婆 / 重置牛 / NTR 开关 | ✅ | ❌ |
| 重置换（失败禁言） | ✅ | ❌ |
| 交换老婆（发起 / 同意 / 拒绝 / 查看） | ✅ | ❌ |
| 管理员命令 | ✅ | ❌ |

- 想要热闹的全套玩法 → **完整版**
- 只想要「每天抽一个 + 抽腻了换一个」，不想有人被抢老婆 / 被禁言 → **Lite 版**

两者配置前缀与数据目录完全隔离，可以同时装在一个 NoneBot2 实例里；
但**指令名相同**，默认会各答一遍。同一实例请**只启用其中一个**，
或给其中一个设 `*_BLOCK_OTHER_PLUGINS=true` 独占。

---

## 快速开始

### 1. 安装

```bash
# 完整版
nb-cli plugin install ./nonebot_plugin_animewifex

# 或 Lite 版
nb-cli plugin install ./nonebot_plugin_animewifex_lite
```

也可以直接把插件目录放进 `src/plugins/` 下启动 NoneBot2。

> 目录名必须以 `nonebot_plugin_` 开头，`nb-cli` 才能管理它。

### 2. 准备图片（本地图库）

两个版本都是**纯本地模式**，不联网取图。请把图片放进对应数据目录：

```text
data/animewifex/img/wife/        ← 完整版
data/animewifex_lite/img/wife/   ← Lite 版
├── 轻小说!明日香.jpg
├── 英雄联盟!阿卡丽.jpg
└── 明日香.jpg
```

文件名建议按 `作品名!角色名.jpg` 命名，机器人会据此播报「来自《作品名》的角色名」；
没有 `!` 就只报角色名。图库为空时会提示获取失败并记一条 warning。

**图库来源（二选一或自备）：**

1. 原插件配套图库：<https://github.com/monbed/wife>
2. 本插件配套图库：<https://github.com/xiaoyaoyo23333/wife-x-ver>

下载后放进 `<数据目录>/img/wife/` 即可。本地图库优先级最高。

### 3. 配置

在 `.env` 里按前缀配置，例如完整版：

```dotenv
ANIMEWIFEX_ADMINS=["123456789"]
ANIMEWIFEX_CHANGE_MAX_PER_DAY=3
ANIMEWIFEX_TIMEZONE=Asia/Shanghai
ANIMEWIFEX_DATA_DIR=D:/nonebot-bot/data/animewifex
```

Lite 版把前缀换成 `ANIMEWIFEX_LITE_`。完整配置项见各自 README。

完整配置示例：

```dotenv
ANIMEWIFEX_ADMINS=["123456789"]      # 管理员 QQ 号列表
ANIMEWIFEX_NEED_PREFIX=false         # 是否需要命令前缀触发
ANIMEWIFEX_NTR_MAX=3                 # 每日牛老婆次数
ANIMEWIFEX_NTR_POSSIBILITY=0.2       # 牛老婆成功率
ANIMEWIFEX_CHANGE_MAX_PER_DAY=3      # 每日换老婆次数
ANIMEWIFEX_SWAP_MAX_PER_DAY=2        # 每日交换请求次数
ANIMEWIFEX_RESET_MAX_USES_PER_DAY=3  # 每日重置机会次数
ANIMEWIFEX_RESET_SUCCESS_RATE=0.3    # 重置成功率
ANIMEWIFEX_RESET_MUTE_DURATION=300   # 重置失败禁言秒数
ANIMEWIFEX_TIMEZONE=Asia/Shanghai    # 判定「今天」的时区
ANIMEWIFEX_COMMAND_PRIORITY=0        # 匹配器优先级
ANIMEWIFEX_BLOCK_OTHER_PLUGINS=false # 是否独占本插件命令
ANIMEWIFEX_DATA_DIR=                 # 数据目录，留空则用 <data_dir>/animewifex
```

常见调法：

| 想要的效果 | 配置 |
| --- | --- |
| 娱乐群，放开牛 | `ANIMEWIFEX_NTR_POSSIBILITY=0.8` + `ANIMEWIFEX_NTR_MAX=10` |
| 禁止赌博 / 禁言太狠 | `ANIMEWIFEX_RESET_SUCCESS_RATE=1.0`（必成）或 `ANIMEWIFEX_RESET_MUTE_DURATION=0`（失败无痛） |
| 熊孩子群 | `ANIMEWIFEX_CHANGE_MAX_PER_DAY=10` + `ANIMEWIFEX_SWAP_MAX_PER_DAY=5` |

> 配置优先级：默认值 < NoneBot2 插件配置机制 < `.env` 的 `ANIMEWIFEX_*` < 环境变量的 `ANIMEWIFEX_*`。
> 这里显式实现了前缀读取，因为 NoneBot2 的 `get_plugin_config` 按字段名直接读环境变量
> （如 `DATA_DIR`），既没有插件前缀，也会被全局同名配置项干扰。

---

## 指令一览

### 完整版

```
【基础命令】
• 抽老婆              每天一次，随机抽一张二次元老婆
• 查老婆 @用户        查看别人的老婆（也支持昵称匹配）

【牛老婆功能】（概率较低 😭）
• 牛老婆 @用户        有概率抢走别人的老婆
• 重置牛 @用户        重置牛的次数，失败会被禁言

【换老婆功能】
• 换老婆              丢弃当前老婆换新的
• 重置换 @用户        重置换老婆的次数，失败会被禁言

【交换功能】
• 交换老婆 @用户      向别人发起老婆交换请求
• 同意交换 @发起者    同意交换请求
• 拒绝交换 @发起者    拒绝交换请求
• 查看交换请求        查看当前的交换请求

【管理员命令】
• 切换ntr开关状态     开启 / 关闭牛老婆功能
```

### Lite 版

只保留 `老婆帮助` / `抽老婆` / `查老婆` / `换老婆` 四条。

### 指令是严格匹配的

只接受「命令」本身，或「命令 + 空格 + 参数」：

| 消息 | 响应 |
| --- | --- |
| `换老婆` / `查老婆 明日香` / `牛老婆 @某人` | ✅ |
| `帮我换老婆` / `换老婆啊` / `xxxx换老婆xxxx` / `换老婆。` | ❌ |

聊天里随口提到「换老婆」不会误触发。

---

## 共存与优先级

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
- **跨版本兼容**：NoneBot2 2.2 ~ 2.5 通用
- **不抢别人消息**：严格指令匹配 + `block=False`，与其它插件互不干扰

---

## 兼容环境

| NoneBot2 | OneBot 适配器 | pydantic | 结果 |
| --- | --- | --- | --- |
| 2.4.2 | 2.4.6 | 2.13 | ✅ 25 项功能测试通过 |
| 2.4.2 | 2.4.3 | 2.13 | ✅ 25 项功能测试通过 |
| 2.5.0 | 2.4.6 | 2.13 | ✅ 25 项功能测试通过（未来升级版本） |

Windows 11 23H2 + Python 3.11.9 实测无差异：插件源码按 Python 3.9+ 语法编写，
未使用 3.12 独有特性。

### 兼容性实现方式

插件通过「能力探测 + 逐级回退」的方式编写，不绑定任何特定 NoneBot2 / 适配器版本：

| 能力 | 处理方式 |
| --- | --- |
| bot / event 获取 | 一律用依赖注入（`bot: Bot, event: Event`），不碰 `matcher.bot` / `matcher.event`——后者在 2.4+ 已不存在 |
| 机器人自身账号 | `bot.self_id`，不依赖 `Event.get_self_id()`（2.4+ 已移除） |
| 群聊判定 | `get_session_type()` → 事件 `message_type` 字段 → `group_id` 属性 |
| 发送者昵称 | `get_user_name()` → `sender.card` / `sender.nickname` → `member.card` / `user.display_name` |
| 消息段构造 | 从事件取适配器的消息段类再调其 `text()` / `at()` / `image()`，缺失则按标准字段直接构造 |
| @ 目标解析 | 按 `segment.type == "at"` + `data["qq"]` 解析，不使用适配器特有的 `is_at()` |
| 禁言 | 依次尝试 `bot.set_group_ban()` / `call_api("set_group_ban")` / `call_api("group_ban")` |
| pydantic | 同时兼容 v1（`__fields__` / `parse_obj_as` / `.copy()`）与 v2（`model_fields` / `TypeAdapter` / `.model_copy()`） |

经实测：**NoneBot2 2.4.2 与 2.5.0 的核心 `Event` / `Matcher` / `MessageSegment` API 形状完全一致**，
均已移除上述便捷方法，因此当前实现对两个版本通用；更早的 2.2 / 2.3 因保留了这些方法，
会走回退链的第一级，同样可用。

---

## Windows 注意事项

1. **时区数据（tzdata）**：Windows 的 Python 默认不带 IANA 时区库，NoneBot2 也不依赖
   `tzdata`。未安装时插件不会崩溃，会退化为固定 UTC+8——由于默认时区本就是东八区，
   「今天」的判断依然正确；但若你把 `ANIMEWIFEX_TIMEZONE` 改成其他时区（如 `America/New_York`
   涉及夏令时的时区），请务必 `pip install tzdata`。
   也可直接用环境变量 `TZ=Asia/Shanghai` 指定（NoneBot2 自身不读取 `TZ`）。

2. **数据目录是相对路径**：默认的 `data/animewifex` 相对于 **NoneBot2 的启动目录（CWD）**。
   如果你用计划任务 / NSSM / 快捷方式启动时 CWD 不是项目根目录，数据会落到别处。
   建议在 `.env` 里显式指定绝对路径：`ANIMEWIFEX_DATA_DIR=D:/nonebot-bot/data/animewifex`。

3. **本地图库文件名**：支持中文与 `!`、`空格` 等字符（`作品名!角色名.jpg`），读取与路径
   校验均按 UTF-8 处理。

---

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

---

## 致谢

- 上游 AstrBot 插件：[astrbot_plugin_animewifex](https://github.com/monbed/astrbot_plugin_animewifex)
- 原版思路：[astrbot_plugin_AW](https://github.com/zgojin/astrbot_plugin_AW)

## License

MIT