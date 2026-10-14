# render-prompt-expert

> 瓷砖 / 室内效果图的**参数化提示词专家**——把参考图或空间需求翻译成可落地的渲染指令，而不是靠形容词碰运气。

[![Type](https://img.shields.io/badge/type-AI--Skill-A8502A)]() [![Domain](https://img.shields.io/badge/领域-瓷砖%20%7C%20室内效果图-2F5D62)]() [![Modules](https://img.shields.io/badge/内置-5类瓷砖PBR%20%2B%206空间预设-4A4640)]()

---

## 解决什么问题

用 AI 做瓷砖效果图，最常翻的三类车：

1. **材质错位** — 瓷砖被画成天然大理石 / 塑料板
2. **尺度错乱** — 纹理大小和真实砖规格对不上
3. **图案重复** — 同一块纹理复制粘贴，一眼假

这三个问题的共同根因是：**提示词里没有可以量化的约束**。本 Skill 的做法是把瓷砖的材质属性翻译成渲染引擎能理解的物理参数（PBR），再加上尺度锚和抗重复指令。

## 内置能力

| 模块 | 内容 |
|------|------|
| **瓷砖 PBR 参数表** | 5 类主流瓷砖（抛光砖 / 哑光砖 / 仿古砖 / 大理石纹砖 / 木纹砖）的 Roughness、Reflection、Clearcoat、Normal 数值区间 + 对应提示词语感 |
| **空间灯光 / 机位预设** | 6 类空间（主卫 / 客厅 / 玄关 / 厨房 / 商业展厅 / 卧室）的灯光方案、色温、机位高度、画面强调点 |
| **翻车规避指令集** | 防过曝、防重复纹理、砖缝对齐、防塑料感、构图清理——出图必带的五条 |
| **复刻模板** | 参考图复刻用的提示词骨架，直接改参数可用 |
| **图生图压缩规则** | ImageGen 请求体超限时的标准处理流程（长边 1024px / JPEG q85 / ≤500KB） |

## 瓷砖 PBR 参数对照（节选）

| 品类 | Roughness | Reflection | Clearcoat | 提示词语感 |
|------|-----------|-----------|-----------|------------|
| 抛光砖 | 0.02–0.08 | 0.6–0.9 | 0.8–1.0 | glossy polished surface, mirror-like reflection, clearcoat |
| 哑光砖 | 0.4–0.7 | 0.1–0.3 | 0–0.2 | matte / satin finish, subtle micro-texture, no specular |
| 仿古砖 | 0.6–0.9 | 0.05–0.2 | 0–0.1 | rustic worn texture, uneven relief, antique |
| 大理石纹砖 | 0.05–0.15 | 0.4–0.7 | 0.6–1.0 | bookmatched veining, clearcoat glaze, translucent grain |
| 木纹砖 | 0.3–0.6 | 0.1–0.3 | 0–0.2 | wood-grain plank, warm tone, soft sheen |

## 空间预设对照（节选）

| 空间 | 灯光 | 色温 | 机位 | 强调 |
|------|------|------|------|------|
| 主卫 | 冷白窗光 + 暖 fill | 5000–6500K | 低机位 0.6–0.9m | 地面光泽 / 对花 |
| 客厅 | 右侧自然光、高 key | 5000–6500K | 35° 俯视 | 大面积地面留白 |
| 商业展厅 | 轨道射灯 + 环境光 | 4000K | 低机位广角 | 砖面整体性 |

## 使用方式

Skill 触发后会自动：
1. 判断场景类型（产品图复刻 / 空间效果图 / 铺贴细节）
2. 查参数表匹配材质与灯光
3. 组装提示词并调用 ImageGen 出图（文生图 / 图生图）

需要手动改参数时，直接查 `references/param-bank.md`。

## 配套 Skill

- **master-style-render** — 大师风格效果图，复用本 Skill 的图生图压缩流程
- **tile-material-params** — 同源的材质参数来源

## 关键原则

- **用毫米值和数值区间，不用形容词** — `1.5mm joints` 有效，`thin joints` 会被忽略
- **尺度锚必须有** — 砖规格 + 缝宽是 AI 判断纹理尺度的唯一依据
- **负向词和正向词同等重要** — 尤其材质类，不写 `not natural stone slab` 就一定会被画成石头

---

## License

MIT
