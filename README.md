# dsh-think-bounce-pet

[📺 点击观看效果演示](https://www.bilibili.com/video/BV14d886GEx1)

DeepSeek Harness Web UI 的「思考弹球宠物」插件。

## 行为

- **常驻球**：插件开启期间，屏幕里常驻一只与 think 状态无关的小球，从随机边外飞入后一直弹跳，不会因为思考结束而消失。
- **think 开始**：当前会话的推理流（`chat.legacy.partial` 出现带文本的 `reasoning` 块）开始时，从视口任意一条边外的随机点发射一只思考球，瞄准界面中部飞入。
- **弹跳**：球在视口内丝滑地弹来弹去（DVD-logo 式直线运动 + 边界反射），每次碰撞按设置的比例损失动能；速度一旦低于最低速度（固定 46 px/s），会把速率回抬到这一档，因此两只球都永远不会停下。两颗球撞到彼此时也会沿碰撞法线互相反弹。
- **Q 弹「ber」**：每次碰壁、点拍、扔出时，球沿运动轴做一次纯缩放的阻尼余弦振荡（~7Hz，约 0.6s 衰减完）；按下抓住的瞬间也会先捏扁，并且按住期间会继续播完这次形变再恢复圆球。全程只用 scale 表达，为后续帧动画化预留。
- **交互**：球在弹跳时都可以用鼠标玩——
  - **点一下**：把球拍走，球以初始动能向「远离点击处」的方向飞出去继续弹；
  - **按住拖动再松手**：朝拖动方向把球扔出去（同样以初始动能）。
- **鼠标碰撞**：鼠标碰撞滑条 `0` 为关闭；`50` 为较大推力（约 400px/s）；`100` 为 `50` 的 5 倍推力（约 2000px/s）。开启后鼠标光标变成一个不可推动的小圆障碍物，球碰到鼠标会按碰撞损耗弹开并受到向外推力，此模式下鼠标不能抓取/拖动球。
- **重力**：设置页重力滑条 `0` 保持现在的无重力弹跳，`50` 为 1 倍向下重力，`100` 为 2 倍向下重力；最低速度保护仍生效，球不会停。
- **think 结束**：推理流消失时，思考球播放缩小+淡出动画后移除；常驻球不受影响。

## 设置（设置页「宠物 · 思考弹球」卡片）

| 参数 | 默认 | 范围 | 说明 |
|---|---|---|---|
| 启用 | 开 | — | 关闭后移除常驻球与思考球，也不再随 think 生成球 |
| 鼠标碰撞 | 0（关） | 0–100 | 0=关闭；50=较大推力（400px/s）；100=50 的 5 倍推力；开启时鼠标为障碍且不能抓取球 |
| 重力 | 0（无） | 0–100 | 0=当前无重力；50=1 倍重力；100=2 倍重力 |
| 初速度 | 1960 px/s | 200–4000 | 发射、拍飞、扔出的初始速率 |
| 碰撞损耗 | 52 % | 0–90 | 每次碰壁损失的速度比例 |
| 摩擦力 | 0（无） | 0–100 | 指数阻尼强度（100 ≈ 每秒速度 ×0.018） |
| 球体大小 | 100 % | 20–200 | 以当前 144px 直径/72px 半径为 100% 基准，实时缩放视觉尺寸与碰撞箱 |

参数改动立即生效（飞行中的球下一帧就按新值计算），持久化在
localStorage 的 `dsh-think-bounce-pet.settings`；v0.1 的旧开关键会自动迁移。
最低速度固定为 46 px/s，不随以上设置项修改；重力模式下贴地时会额外保证竖直方向至少能弹起约 80px 高度，避免球贴地滚/抽搐。

## 结构

标准双面插件：宿主半边 `lib/index.js` 为空壳（纯客户端 UI 插件）；
浏览器半边 `lib/client.js` 通过 `shell.overlay` 槽位挂载全屏
`pointer-events:none` 图层（每颗球各自一个图层，球本体 `pointer-events:auto`），通过
`settings.section` 槽位提供开关与测试按钮（设置存 localStorage）。

## 安装 / 更新

```powershell
# 在插件仓库的父目录执行（即该 README 所在目录的上级目录）
& '.\node_modules\.bin\dsh.cmd' plugin --profile web add './dsh-think-bounce-pet'
& '.\node_modules\.bin\dsh.cmd' plugin --profile web update dsh-think-bounce-pet
```

客户端半边改动只需浏览器 F5；本插件无宿主侧逻辑，装好后重启一次 `dsh web`
即可长期生效。

## 插件市场适配（已发布 GitHub，尚未提交商城）

本插件已按 DSH 插件规范整理成可被插件市场识别/管理的双面包：

- `package.json` 声明了 `dsh.bundle.patch` 与 `dsh.client`（`platform: "web"`），并导出 `./client`；
- `cordis.patch.yml` 提供 bundle 挂载行；
- `lib/index.js` 为宿主空壳，`lib/client.js` 为浏览器半边。

当前仓库状态（已验证）：

- 源码已发布到 GitHub 公开仓库：https://github.com/9livewolf/dsh-think-bounce-pet；
- 已通过本地链接安装到 web profile：`dsh plugin --profile web add ./dsh-think-bounce-pet`；
- 插件市场「本机插件」中会显示 `dsh-think-bounce-pet`（来源类型：本地链接），可停用/启用/查看状态；
- `GET /plugins/dsh-think-bounce-pet/client.js` 返回 200。

要让它出现在公共插件市场的列表里并一键下载安装，需要：
1. 把该仓库改为 Public；
2. 向 awesome-dsh-plugin.com 目录提交收录（例如 PR 到对应 awesome 列表/registry）。

当前仓库已是 Public，尚未提交到商城目录。
