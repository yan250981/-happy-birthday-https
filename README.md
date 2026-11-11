# 岁光 · AGE IN LIGHT

> 一团会听你说话、也会被你的手拨开的星尘。

一个用 **Three.js + GLSL + MediaPipe Hands** 写的生日快乐互动网页。
粒子是页面的唯一主体：它们从散落的星尘聚合成「生日快乐」四个字，
被手势扰动、塑成心形或星星，最后在吹灭蜡烛后炸成烟花。

**在线体验**：https://age-in-light.app.workbuddy.host/?sharecode=KRRLEWA7HM

---

## 特性

| | |
|---|---|
| **GPU 粒子形态变换** | 26000 粒子，单个 `THREE.Points`，靠 `aTargetA`/`aTargetB` + `aMorph` 在顶点着色器内插值完成「散落 → 文字 → 蛋糕 → 轨道环 → 绽放」连续变形，一次 draw call |
| **实时手势扰动** | MediaPipe HandLandmarker 追踪手部，掌心位置作为斥力场推挤粒子 |
| **双路径吹蜡烛** | 快速划过（吹一口气）＋ 停留蓄力（烛焰膨胀后自动熄灭），慢动作也能完成仪式 |
| **捏合塑形彩蛋** | 捏合手势让粒子收拢成心形或星星 |
| **零音频文件** | 吹气、铃音、扫频、烟花轰鸣全部由 WebAudio 实时合成 |
| **零后端** | `?to=&from=&msg=` URL 参数定制 + 一键复制专属链接 |
| **离线可运行** | MediaPipe wasm 与模型全部本地化，运行时不依赖任何 CDN |

## 快速开始

```bash
npm install
npm run dev        # http://127.0.0.1:5173/
npm run build      # 产物输出到 dist/
```

**定制链接**：

```
http://127.0.0.1:5173/?to=小美&from=小刘&msg=今年也要好好吃饭呀
```

## 替换素材

所有素材均为程序化占位，把真实文件放到下列路径即可，缺失时自动降级回占位：

| 路径 | 内容 |
|---|---|
| `public/media/photos/01.jpg` ~ `12.jpg` | 寿星照片，环绕主体公转 |
| `public/media/video/wish.mp4` | 照片墙 / 祝福视频 |
| `public/media/bgm.mp3` | 背景音乐 |
| `public/media/signature.png` | Logo / 手写签名图 |

构建期会扫描 `public/media/` 生成 `media-manifest.json`，运行时查表判断素材是否就位 ——
这样既不会产生 404 日志，也不会为缺失素材付出请求开销。

## 设备策略

这是一个**桌面作品**。

- **桌面端**：完整手势版。点击「开启手势魔法」授权摄像头后可用手拨粒子、捏合塑形、吹灭蜡烛。
- **手机端**：自动进入**观礼版** —— 无需摄像头权限，粒子动画、章节推进、烟花与祝福揭晓完整保留。
- 摄像头授权失败或被拒绝时，静默降级为指针驱动，交互逻辑完全一致。

## 项目结构

```
src/
  main.js                  主编排：状态机、章节调度、吹灭与庆祝逻辑
  scene/
    World.js               渲染器 / 相机 / 后期链 / 蜡烛 / 烟花 / 轨迹
    ParticleField.js       主体粒子系统（形态变换 + 手部斥力场）
    shapes.js              形态生成器：文字采样 / 散落 / 环 / 蛋糕 / 心 / 星
    shaders.js             全部 GLSL：背景、粒子、烟花、彩带、烛焰、蛋糕
  interaction/
    HandInput.js           MediaPipe 手势识别（捏合 / 张开 / 骨架绘制）
    PointerInput.js        指针降级输入（速度检测 + 蓄力吹气）
  ui/
    UI.js                  章节导视 / 提示 / 封面 / 结尾卡片 / 设置
    PhotoLayer.js          照片卡的椭圆轨道公转
  audio/Sfx.js             WebAudio 实时音效合成
  assets/media.js          素材清单与占位生成
public/mediapipe/          本地化的 wasm 与手部模型
scripts/                   Playwright 验证脚本
```

## 验证

```bash
npm run verify        # 全流程 + 吹灭双路径 + 附加功能
npm run verify:prod   # 生产构建 + 冒烟测试
```

脚本基于 Playwright + 无头 Chrome，验证封面、章节推进、吹灭蜡烛、塑形彩蛋、
保存成品图、复制链接、手机端观礼版，并统计控制台错误。

## 设计说明

参考「星屑成诗 STARLIGHT BIRTHDAY」的 8 项设计语言分析与取舍说明见 [`DESIGN.md`](./DESIGN.md)。

## 技术要点

- **动画一律用真实时间驱动**（`performance.now()`），不用 clamped `dt` 累加 ——
  否则低帧率设备上整个动画会变慢。
- **粒子自转必须整体同步**：每个粒子用不同角度自转会把「生日快乐」打散成云团。
- **文字转点阵**靠离屏 Canvas 采样 alpha + 亮度双通道判定，避免抗锯齿边缘导致采样点偏移。

## 授权

MIT
