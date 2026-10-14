---
name: render-prompt-expert
version: 1.0.0
description: 瓷砖/室内效果图提示词专家。把参考图或空间需求翻译成可落地的渲染提示词，并调用 ImageGen 出图（文生图 + 图生图）。内置材质参数映射、空间灯光机位预设、图生图压缩规则，对齐 tile-material-params Skill。
trigger:
  - 怎么做这样的效果图
  - 复刻参考图
  - 生成瓷砖效果图提示词
  - 室内效果图渲染提示词
  - 出一张瓷砖渲染图
  - render prompt for tiles
  - 效果图提示词
---

# render-prompt-expert（效果图提示词专家）

把「参考图 / 空间需求」转化为可执行的瓷砖/室内高清效果图提示词，并直接出图。
核心原则：**参数驱动、可复用、风格可控**——不空泛写提示词，而是从材质参数表反推质感描述。

## 何时使用

- 用户发来一张参考图，问「怎么做成这样的效果图」
- 用户要某空间 + 某瓷砖品类的高清渲染图
- 需要批量产出「空间 × 品类」提示词用于 ImageGen / ComfyUI / Blender

## 工作流（四步）

### 1. 解析参考图 / 需求
- 若用户提供图片：先看清 **瓷砖品类**（抛光/哑光/仿古/大理石纹/木纹）、**灯光**（高 key 柔光 / 冷暖对比 / 局部射灯）、**构图**（机位、留白、主体）、**软装元素**。
- 若用户提供文字需求：提取「空间 + 瓷砖品类 + 风格倾向」。
- 方向描述一律用 **上/下/左/右**，不用东南西北。

### 2. 映射材质参数（对齐 tile-material-params Skill）
从品类定位 PBR 区间（详见 `references/param-bank.md`）：

| 品类 | Roughness | Reflection | Clearcoat | Normal |
|------|-----------|-----------|-----------|--------|
| 抛光砖 | 0.02–0.08 | 0.6–0.9 | 0.8–1.0 | 0.1–0.3 |
| 哑光砖 | 0.4–0.7 | 0.1–0.3 | 0–0.2 | 0.2–0.4 |
| 仿古砖 | 0.6–0.9 | 0.05–0.2 | 0–0.1 | 0.4–0.8 |
| 大理石纹砖 | 0.05–0.15 | 0.4–0.7 | 0.6–1.0 | 0.1–0.3 |
| 木纹砖 | 0.3–0.6 | 0.1–0.3 | 0–0.2 | 0.2–0.5 |

把这些区间翻译成提示词语感：
- 低粗糙度 → "glossy / polished / clearcoat surface"
- 高粗糙度 → "matte / satin / subtle micro-texture"
- 强反射 → "strong reflection, catches light"
- 高清漆 → "clearcoat glaze, translucent grain"
- 低法线 → "only faint relief, no plastic look"

### 3. 组装提示词
固定结构（中文描述用户可读，英文直投 ImageGen）：
```
[画质] + [空间] + [瓷砖材质质感] + [灯光色温] + [构图机位] + [软装] + [风格]
```
- 画质：`Photorealistic high-definition 8k sharp detail`
- 灯光：`soft natural daylight from the right side / bright high-key clean lighting / warm-cool balanced`
- 机位：`low camera angle emphasizing floor / 35° top-down / eye-level 1.5m`
- 风格：`commercial tile product photography / architectural photography style`

参考图复刻时额外加 `in the style of the reference, keep material continuity`。

### 4. 出图（ImageGen）
- **文生图**：直接投英文 prompt。
- **图生图（参考图复刻）**：用 `image` 参数传本地路径，`input_fidelity="high"`，prompt 描述目标。
- 尺寸：竖图 `1024x1536`，横图 `1536x1024`，方图 `1024x1024`。质量 `high`。
- 输出目录：默认工作区 `examples/`，同时复制到桌面供用户查看。

#### ⚠️ 图生图压缩规则（必看）
ImageGen 图生图接口有 **请求体大小上限**，原图 > 约 1MB 极易触发
`Request body larger than maxBodyLength limit`。
处理：生成前先把参考图压到 **长边 1024px、JPEG quality 85、目标 ≤ 500KB**：

```python
from PIL import Image
im = Image.open(src).convert('RGB')
w = 1024
h = int(im.height * w / im.width)
im = im.resize((w, h), Image.LANCZOS)
im.save(dst, 'JPEG', quality=85, optimize=True)
```

再用压缩后的图作为 `image` 输入。

## 配套 Skill
- `tile-material-params`（专家 interior-render-expert 下）：5 类瓷砖完整 PBR 参数 + 翻车规避清单，本 Skill 的参数表与之同源。
- `interior-render-expert`：渲染专家角色，可直接调用本 Skill 带参数出图。

## 输出规范
- 文件命名语义化：`空间_瓷砖品类_风格.png`（如 `主卫_大理石纹砖_暖光.png`）。
- 生成后复制到桌面，并回传「参数对照表 + 所用 prompt」。

## 信用点提示
每次 ImageGen 出图约消耗 **5–10 信用点**，图生图同价；批量前先告知用户总量。
