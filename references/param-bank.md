# 效果图提示词参数库（param-bank）

与 `tile-material-params` Skill 同源。本文件供 `render-prompt-expert` 直接查表组装提示词。

## 一、瓷砖 PBR 参数总表

| 品类 | Roughness | Reflection | Clearcoat | Normal | 提示词语感 |
|------|-----------|-----------|-----------|--------|------------|
| 抛光砖 | 0.02–0.08 | 0.6–0.9 | 0.8–1.0 | 0.1–0.3 | glossy polished surface, mirror-like reflection, clearcoat, very low roughness |
| 哑光砖 | 0.4–0.7 | 0.1–0.3 | 0–0.2 | 0.2–0.4 | matte / satin finish, subtle micro-texture, no specular, diffuse |
| 仿古砖 | 0.6–0.9 | 0.05–0.2 | 0–0.1 | 0.4–0.8 | rustic worn texture, uneven relief, antique, tactile surface |
| 大理石纹砖 | 0.05–0.15 | 0.4–0.7 | 0.6–1.0 | 0.1–0.3 | bookmatched veining, clearcoat glaze, translucent grain, smooth |
| 木纹砖 | 0.3–0.6 | 0.1–0.3 | 0–0.2 | 0.2–0.5 | wood-grain plank, warm tone, soft sheen, natural grain |

## 二、空间灯光 / 机位预设

| 空间 | 灯光 | 色温 | 机位 | 强调 |
|------|------|------|------|------|
| 主卫 | 冷白窗光 + 暖 fill | 5000–6500K | 低机位 0.6–0.9m | 地面光泽 / 对花 |
| 客厅 | 右侧自然光、高 key | 5000–6500K | 35° 俯视 | 大面积地面留白 |
| 玄关 | 顶部柔光 + 局部射灯 | 3000–4000K | 平视 1.4–1.6m | 入口质感 |
| 厨房 | 均匀白光 | 5000–6500K | 俯视 45° | 易洁面反光 |
| 商业展厅 | 轨道射灯 + 环境光 | 4000K | 低机位广角 | 砖面整体性 |
| 卧室 | 暖调柔光 | 2700–3500K | 平视 | 温润氛围 |

## 三、通用翻车规避（出图提示词必带）
- 防过曝：`balanced lighting, no blown highlights`
- 防重复纹理：`non-repeating texture, large tile pattern`
- 砖缝对齐：`continuous grout lines, aligned tiles`
- 材质不发塑料：`realistic ceramic material, not plastic`
- 构图干净：`no clutter, minimal styling`

## 四、参考图复刻提示词模板（可直接改）
```
Photorealistic interior product rendering, minimalist modern [空间] close-up,
[瓷砖颜色] [品类语感] tile floor with [肌理描述],
[灯光描述], bright high-key clean lighting, low contrast,
[软装元素，含上/下/左/右位置],
elegant and serene atmosphere, commercial tile photography, 8k, sharp detail.
```

## 五、图生图压缩脚本（ImageGen 请求体超限时用）
```python
from PIL import Image
im = Image.open(src).convert('RGB')
w = 1024
h = int(im.height * w / im.width)
im = im.resize((w, h), Image.LANCZOS)
im.save(dst, 'JPEG', quality=85, optimize=True)
# 目标文件 ≤ 500KB，再作 image 参数传入
```
