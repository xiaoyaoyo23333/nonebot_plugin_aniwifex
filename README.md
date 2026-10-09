# nonebot_plugin_animewifex

二次元老婆插件的 **NoneBot2** 版本，由 AstrBot 插件
[astrbot_plugin_animewifex](https://github.com/monbed/astrbot_plugin_animewifex) 移植而来。

命令、业务逻辑、数据格式与持久化方式与 AstrBot 版保持一致。

使用Minimax M3.1-Flash-Preview进行辅助

## 安装

```bash
nb-cli plugin install ./nonebot_plugin_animewifex
```

或直接把整个目录复制到 `src/plugins/animewifex/` 后启动 NoneBot2。

依赖：

- `aiohttp` —— NoneBot2 本身已依赖，通常无需处理。
- `tzdata` —— **Windows 上建议安装**（见下方「Windows 注意事项」）。不装也能运行，
  插件会自动回退到固定 UTC+8。安装方式：`pip install tzdata`。

## 已验证的运行环境

| NoneBot2 | OneBot 适配器 | pydantic | 结果 |
| --- | --- | --- | --- |
| 2.4.2 | 2.4.6 | 2.13 | ✅ 25 项功能测试通过 |
| 2.4.2 | 2.4.3 | 2.13 | ✅ 25 项功能测试通过 |
| 2.5.0 | 2.4.6 | 2.13 | ✅ 25 项功能测试通过（未来升级版本） |

Windows 11 23H2 + Python 3.11.9 实测无差异：插件源码按 Python 3.9+ 语法编写，
未使用 3.12 独有特性。

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

## 配置

在 `.env` 或系统环境变量中配置，插件级配置以 `ANIMEWIFEX_` 为前缀：

```dotenv
ANIMEWIFEX_ADMINS=["123456789"]      # 管理员 QQ 号列表
ANIMEWIFEX_NEED_PREFIX=false        # 是否需要命令前缀触发
ANIMEWIFEX_NTR_MAX=3                # 每日牛老婆次数
ANIMEWIFEX_NTR_POSSIBILITY=0.2      # 牛老婆成功率
ANIMEWIFEX_CHANGE_MAX_PER_DAY=3     # 每日换老婆次数
ANIMEWIFEX_SWAP_MAX_PER_DAY=2       # 每日交换请求次数
ANIMEWIFEX_RESET_MAX_USES_PER_DAY=3 # 每日重置机会次数
ANIMEWIFEX_RESET_SUCCESS_RATE=0.3   # 重置成功率
ANIMEWIFEX_RESET_MUTE_DURATION=300  # 重置失败禁言秒数
ANIMEWIFEX_IMAGE_BASE_URL=https://cdn.jsdmirror.com/gh/monbed/wife@main
ANIMEWIFEX_IMAGE_LIST_URL=https://animewife.dpdns.org/list.txt
ANIMEWIFEX_TIMEZONE=Asia/Shanghai   # 判定「今天」用的时区；留空则依次回退 NoneBot2 全区 timezone、TZ、Asia/Shanghai
ANIMEWIFEX_COMMAND_PRIORITY=0       # 匹配器优先级，越小越先执行（详见下方「与其它老婆插件共存」）
ANIMEWIFEX_BLOCK_OTHER_PLUGINS=false # true=独占本插件命令；false=其它老婆插件仍可各自回应
ANIMEWIFEX_DATA_DIR=                # 数据目录，留空则用 <data_dir>/animewifex
```

常见调法：

| 想要的效果 | 配置 |
| --- | --- |
| 娱乐群，放开牛 | `ANIMEWIFEX_NTR_POSSIBILITY=0.8` + `ANIMEWIFEX_NTR_MAX=10` |
| 禁止赌博 / 禁言太狠 | `ANIMEWIFEX_RESET_SUCCESS_RATE=1.0`（必成）或 `ANIMEWIFEX_RESET_MUTE_DURATION=0`（失败无痛） |
| 熊孩子群 | `ANIMEWIFEX_CHANGE_MAX_PER_DAY=10` + `ANIMEWIFEX_SWAP_MAX_PER_DAY=5` |
| 抽不到图时手动补图 | 往 `<数据目录>/img/wife/` 放 `作品名!角色名.jpg`，本地图库优先级最高 |

> 配置优先级：默认值 < NoneBot2 插件配置机制 < `.env` 的 `ANIMEWIFEX_*` < 环境变量的 `ANIMEWIFEX_*`。
> 这里显式实现了前缀读取，因为 NoneBot2 的 `get_plugin_config` 是按字段名直接读环境变量
> （如 `DATA_DIR`），既没有插件前缀，也会被全局同名配置项干扰。

图床
1.原插件配套图库：
（配套仓库 https://github.com/monbed/wife ）按网络环境二选一：
- 能直连 GitHub：`ANIMEWIFEX_IMAGE_BASE_URL=https://raw.githubusercontent.com/monbed/wife/main/`
- 用反代：`https://fastly.jsdelivr.net/gh/monbed/wife@main/` 或 `https://cdn.jsdmirror.com/gh/monbed/wife@main/`

2.下载本插件配套的图库（需放在本地）
[xiaoyaoyo23333/wife-x-ver](https://github.com/xiaoyaoyo23333/wife-x-ver)

（也可以手动下载图片放进 `<数据目录>/img/wife/`，插件会优先从本地图库抽取，
文件名建议 `作品名!角色名.jpg`：

```text
data/animewifex/img/wife/
├── 轻小说!明日香.jpg
├── 英雄联盟!阿卡丽.jpg
└── 明日香.jpg      ← 没有 ! 的也能用，只是文案不报作品
```

取图优先级：**本地图库 → 有效缓存（1 小时内）→ 远程列表 → 过期缓存兜底**，
远程挂了也不会完全抽不出图。

> **前缀触发**：把 `ANIMEWIFEX_NEED_PREFIX` 设为 `true` 后，命令必须以 `COMMAND_START` 中的前缀
> 开头（如 `/抽老婆`）。若 `.env` 里的 `COMMAND_START` 含空字符串（默认「所有消息都视为命令」），
> 前缀模式不会生效，请先去掉空字符串。

> **管理员**：`ANIMEWIFEX_ADMINS` 与 NoneBot2 的 `SUPERUSERS` 合并生效，
> 两者都能使用「切换ntr开关状态」和无条件「重置牛 / 重置换」。

## 功能与用法

群聊每天重置的「二次元老婆」玩法，13 条命令，**仅在群聊生效**，私聊不触发。

四个独立玩法：**抽老婆**（基础）、**牛老婆**（抢别人的）、**换老婆**（重抽）、
**交换老婆**（互换）；外加一套带赌博性质的重置机制，以及管理员开关。

### 指令速查

| 指令 | 作用 | 限制 |
| --- | --- | --- |
| `老婆帮助` | 列出全部命令 | — |
| `抽老婆` | 抽今天的老婆 | 每天一次，当天不再变 |
| `查老婆 [@用户]` / `查老婆 昵称` | 查自己或别人的老婆 | — |
| `牛老婆 @用户` / `牛老婆 昵称` | 有概率抢走对方老婆 | 每天 3 次，成功率 20% |
| `重置牛 @用户` | 重置牛次数（失败禁言） | 每天 3 次机会，成功率 30% |
| `换老婆` | 丢弃当前老婆重抽 | 每天 3 次，需已有老婆 |
| `重置换 @用户` | 重置换次数（失败禁言） | 与 `重置牛` **共享**次数 |
| `交换老婆 @用户` | 发起互换请求 | 每天 2 次，双方都需有老婆 |
| `同意交换 @发起者` | 接受请求 | 跨天自动过期 |
| `拒绝交换 @发起者` | 拒绝请求 | — |
| `查看交换请求` | 查看自己相关的请求 | — |
| `切换ntr开关状态` | 开关牛老婆 | 仅管理员 |

### 抽老婆 / 查老婆

```
小明：抽老婆
Bot ：小明，你今天的老婆是来自《轻小说》的明日香，请好好珍惜哦~ [图片]
```

之后一整天再发 `抽老婆`，返回的是同一个人（幂等）。

```
小明：查老婆
Bot ：小明的老婆是来自《轻小说》的明日香，羡慕吗？ [图片]

小明：查老婆 @小红
Bot ：小红的老婆是来自《单作者》的凉宫，羡慕吗？ [图片]

小明：查老婆 凉宫          ← 不 @ 也可以，昵称匹配
Bot ：小明的老婆是来自《单作者》的凉宫，羡慕吗？ [图片]
```

> **图片名决定文案**：图库文件名按 `作品名!角色名.jpg` 命名，`!` 前当作作品来源；
> 没有 `!` 就只报角色名。

### 牛老婆（概率玩法）

需要对方**今天已经有老婆**。每次消耗一次机会，成功则老婆直接归你。

```
小明：牛老婆 @小红
Bot ：小明，牛老婆成功！老婆已归你所有，恭喜恭喜~
      小明，你今天的老婆是来自《轻小说》的明日香，请好好珍惜哦~ [图片]
```

各类拒绝回复：

```
牛老婆（没 @ 人）    → 小明，请@你想牛的对象，或输入完整的昵称哦~
牛老婆 @自己        → 小明，不能牛自己呀，换个人试试吧~
牛老婆 @小红（超限） → 小明，你今天已经牛了3次啦，明天再来吧~
牛老婆 @小红（没抽） → 对方今天还没有老婆可牛哦~
牛老婆 @小红（失败） → 小明，很遗憾，牛失败了！你今天还可以再试2次~
```

成功后**对方当天老婆直接消失**，得自己重新抽。

### 换老婆

```
小明：换老婆
Bot ：小明，你今天的老婆是来自《相合之物》的雪平一果，请好好珍惜哦~ [图片]
```

- 当天还没抽过老婆会被拦：`小明，你今天还没有老婆，先去抽一个再来换吧~`
- 抽取失败时老婆和次数都保持不动，不会白扣次数。

### 交换老婆（最完整的玩法）

```
小明：交换老婆 @小红
Bot ：小明 想和 @小红 交换老婆啦！请对方用"同意交换 @发起者"或"拒绝交换 @发起者"来回应~

小红：查看交换请求
Bot ：当前交换请求如下：
      → 小明 发起给你的交换请求
      请在"同意交换"或"拒绝交换"命令后@发起者进行操作~

小红：同意交换 @小明
Bot ：交换成功！你们的老婆已经互换啦，祝幸福~

—— 或者 ——

小红：拒绝交换 @小明
Bot ：@小明，对方婉拒了你的交换请求，下次加油吧~
```

几个容易踩的点：

- **互相 @ 的顺序**：发起方 `@对方`，回应方 `@发起者`，反了就提示
  `小明，请在命令后@发起者，或用"查看交换请求"命令查看当前请求哦~`。
- **换目标会退还次数**：直接对别人重新发 `交换老婆`，会提示
  `已自动取消你之前发起的交换请求并返还次数~`，不会因为次数用完被卡住。
- **老婆变了请求就作废**：任何一方 `换老婆` 或 `牛老婆` 成功后，相关请求自动取消并退还次数，
  提示 `已自动取消 N 条相关的交换请求并返还次数~`；若双方老婆都已变化则提示
  `交换失败，有一方的老婆已经发生变化，请重新发起交换吧~`。
- **跨天失效**：零点后请求过期，回应时提示 `该交换请求已过期，请重新发起交换吧~`。

### 重置牛 / 重置换（带赌博）

```
小明：重置牛 @小明
Bot ：已重置 @小明 的牛老婆次数。              ← 成功

小明：重置换 @小明
Bot ：小明，重置换失败，被禁言300秒，下次记得再接再厉哦~   ← 失败，直接禁言
```

- 每天共 3 次机会，`重置牛` 和 `重置换` **共享**这 3 次。
- 成功后只是把目标（`@某人`）的次数清零，因此可以用在自己身上把次数要回来。
- 次数用完：`小明，你今天已经用完3次重置机会啦，明天再来吧~`

### 管理员

配置在 `ANIMEWIFEX_ADMINS` 里的 QQ，以及 NoneBot2 的 `SUPERUSERS`，都能使用：

```
管理员：切换ntr开关状态
Bot ：小明，NTR已关闭

管理员：重置牛 @小明
Bot ：管理员操作：已重置 @小明 的牛老婆次数。      ← 无视成功率，也不消耗次数

小明：牛老婆 @小红
Bot ：牛老婆功能还没开启哦，请联系管理员开启~
```

NTR 开关**按群保存**，关掉只影响当前群。

## 与其它老婆插件共存

这类「老婆」插件（`nonebot_plugin_today_waifu`、各种今日老婆类插件）几乎都注册了
`抽老婆` / `换老婆`，而 `on_message` 默认 `block=True`——**谁先匹配到，事件就不再往下传**，
排在后面的插件会「静默无响应」，日志里只能看到
`Event will be handled by Matcher(... module=别的插件 ...)`。

本插件的做法是**只认自己的命令，答完放行**：

| 配置项 | 默认 | 作用 |
| --- | --- | --- |
| `ANIMEWIFEX_COMMAND_PRIORITY` | `0` | 排在通用 `on_message` 插件之前，避免被对方 `block` 吞掉 |
| `ANIMEWIFEX_BLOCK_OTHER_PLUGINS` | `false` | 答完后**放行**，对方插件仍可对同一条指令各自回应 |
| 匹配器 `rule` | — | 仅消息以本插件命令开头时触发，不响应普通聊天 |

所以**默认状态下可以和其它老婆插件共存**，各答各的。典型用法：

```
群友：抽老婆
Bot ：小明的群友老婆是 @张三~          ← today_waifu（抽群友）
Bot ：小明，你今天的老婆是来自《Vtuber》的白波来梦~  ← 本插件（抽二次元）
```

若你只想让二次元老婆生效、不想被别人抢答，把 `ANIMEWIFEX_BLOCK_OTHER_PLUGINS` 设为
`true`，本插件就会拦下后续插件，同一条指令只由它回答。

> ⚠️ 别把 `ANIMEWIFEX_COMMAND_PRIORITY` 调大：对方若是 `block=True` 且优先级比你低，
> 你就又收不到事件了（这正是老版本 `priority=99` 时「换老婆没反应」的原因）。

> 配置在插件加载时读取，**修改后需重启 NoneBot2** 才会生效。

## 常见排查

| 现象 | 处理 |
| --- | --- |
| 插件加载报错 / 时区相关异常 | `pip install tzdata`（Windows 上必需） |
| 数据好像清零了 | 检查 NoneBot2 的启动 CWD，建议 `ANIMEWIFEX_DATA_DIR` 写绝对路径 |
| 抽不出图 | 看日志有无拉取失败；先往 `img/wife/` 放两张本地图验证链路 |
| 指令没反应 | 必须在群里发（私聊不触发）；若开了 `NEED_PREFIX`，得用 `/抽老婆` |
| **某条命令单独没反应、日志显示 `module=` 别的插件** | 优先级被对方 `block` 抢走了。保持 `ANIMEWIFEX_COMMAND_PRIORITY=0`（默认）即可，见上方「与其它老婆插件共存」 |
| 同一个指令被回了两遍 | 想要独占就把 `ANIMEWIFEX_BLOCK_OTHER_PLUGINS=true`；想共存就保持默认 `false` |
| **插件目录叫 `noneb_plugin_xxx`** | 拼写少了 `ot`。本地 `plugins/` 目录能正常加载，但 `nb-cli` 的安装/卸载要求目录名以 `nonebot_plugin_` 开头，建议改名 |
| 文案不报作品名 | 图库文件名去掉 `!` 前缀即可 |
| 换到了奇怪的老婆 | 检查 `ANIMEWIFEX_IMAGE_LIST_URL` 指向的列表内容 |

## 数据存储

数据目录默认为 `<NoneBot2 data_dir>/animewifex`（全局 `data_dir` 未配置时为
`<项目目录>/data/animewifex`），可用 `ANIMEWIFEX_DATA_DIR` 覆盖：

```text
animewifex/
├── config/
│   ├── <群号>.json          # 该群成员的老婆记录
│   ├── records.json         # 每日次数记录（ntr / change / reset / swap）
│   ├── swap_requests.json   # 交换请求
│   ├── ntr_status.json      # 各群 NTR 开关
│   └── wife_list_cache.txt  # 图片列表缓存
└── img/wife/                # 本地图库（可手动放图）
```

JSON 均为原子写入（临时文件 + 替换），并对文件被外部改坏的情况做了降级处理。

## 与 AstrBot 版的差异

- 事件监听由 `@filter.event_message_type(GROUP_MESSAGE)` 改为带 `rule` 的 `on_message`：
  只在消息以本插件命令开头时触发，答完默认放行（`block=False`），可与其它老婆插件共存，
  见上方「与其它老婆插件共存」。
- 「触发前缀」由 NoneBot2 的 `COMMAND_START` 实现（见上方说明）。
- 管理员判定增加了 NoneBot2 `SUPERUSERS`。
- 禁言调用兼容 OneBot V11 / V12 的 API 形式，失败时静默忽略。
- 时区改读 `ANIMEWIFEX_TIMEZONE`（回退到 NoneBot2 的 `timezone`、`TZ` 环境变量，
  最后回退 `Asia/Shanghai`）。

## 兼容性

插件通过「能力探测 + 逐级回退」的方式编写，不绑定任何特定 NoneBot2 / 适配器版本：

| 能力 | 处理方式 |
| --- | --- |
| bot / event 获取 | 一律用依赖注入（`bot: Bot, event: Event`），不碰 `matcher.bot` / `matcher.event`——后者在 2.4+ 已不存在 |
| 机器人自身账号 | `bot.self_id`，不依赖 `Event.get_self_id()`（2.4+ 已移除） |
| 群聊判定 | `get_session_type()` → 事件 `message_type` 字段 → `group_id` 属性 |
| 发送者昵称 | `get_user_name()` → `sender.card` / `sender.nickname` → `member.card` / `user.display_name` |
| 消息段构造 | 从事件取适配器的消息段类再调其 `text()` / `at()` / `image()`，缺失则按标准字段直接构造——核心基类在 2.4+ 已不再提供这些工厂方法 |
| @ 目标解析 | 按 `segment.type == "at"` + `data["qq"]` 解析，不使用适配器特有的 `is_at()` |
| 禁言 | 依次尝试 `bot.set_group_ban()` / `call_api("set_group_ban")` / `call_api("group_ban")` |
| pydantic | 同时兼容 v1（`__fields__` / `parse_obj_as` / `.copy()`）与 v2（`model_fields` / `TypeAdapter` / `.model_copy()`） |

经实测：**NoneBot2 2.4.2 与 2.5.0 的核心 `Event` / `Matcher` / `MessageSegment` API 形状完全一致**，
均已移除上述便捷方法，因此当前实现对两个版本通用；更早的 2.2 / 2.3 因保留了这些方法，
会走回退链的第一级，同样可用。
