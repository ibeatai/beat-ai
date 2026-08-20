# BeatAI Logo Files

## Logo Overview

BeatAI Logo使用音频波形设计，代表"Beat"的节拍感和AI的智能。

### 设计元素
- 7个波形条：形成律动的节拍视觉
- 渐变配色：Classic Blue主题色 (#3b82f6 → #60a5fa)
- 装饰点：增强设计细节

## Generated Files

### SVG (矢量图)
- `beatai-logo-classic-blue.svg` - 原始SVG文件，无损缩放

### PNG (位图)
- `beatai-logo-32.png` - 32x32px (993B) - 适用于小图标
- `beatai-logo-64.png` - 64x64px (2.3KB) - 适用于中等图标
- `beatai-logo-128.png` - 128x128px (5.8KB) - 适用于大图标
- `beatai-logo-256.png` - 256x256px (17KB) - 适用于高清图标
- `beatai-logo-512.png` - 512x512px (56KB) - 适用于超高清显示
- `beatai-logo-classic-blue.png` - 512x512px (56KB) - 主Logo文件

## Usage

### In React Components
```jsx
import logo from './beatai-logo-512.png';

<img src={logo} alt="BeatAI" width="64" />
```

### In HTML
```html
<link rel="icon" href="/beatai-logo-32.png" sizes="32x32">
<link rel="icon" href="/beatai-logo-64.png" sizes="64x64">
<link rel="apple-touch-icon" href="/beatai-logo-256.png">
```

### As Favicon
```html
<link rel="icon" type="image/png" href="/beatai-logo-32.png">
```

## Color Theme

**Classic Blue**
- Primary: #3b82f6
- Light: #60a5fa
- Gradient: linear-gradient(180deg, #3b82f6 0%, #60a5fa 100%)

## Generation

Files generated using:
```bash
node generate-logo.js
bash generate-logo-png.sh
```

Or manually via qlmanage:
```bash
qlmanage -t -s 512 -o /tmp public/beatai-logo-classic-blue.svg
```

## License

© 2024 BeatAI. All rights reserved.
