# 粒子特效设计指南

## 1. 贴图资源获取

### 樱花花瓣贴图
- **推荐网站**：
  - [Unsplash](https://unsplash.com/s/photos/cherry-blossom-petal) - 免费高质量图片
  - [Pexels](https://www.pexels.com/search/cherry%20blossom/) - 免费图片
  - [Pixabay](https://pixabay.com/images/search/cherry%20blossom/) - 免费图片
  - [Flaticon](https://www.flaticon.com/search?word=cherry%20blossom) - 图标资源

- **搜索关键词**：
  - "cherry blossom petal"
  - "sakura petal"
  - "cherry flower petal"
  - "樱花花瓣"

- **图片要求**：
  - 格式：PNG（透明背景）
  - 尺寸：建议 32x32px 到 64x64px
  - 背景：透明
  - 数量：建议准备 3-5 种不同形状/角度的花瓣

### 落叶贴图
- **推荐网站**：
  - [Unsplash](https://unsplash.com/s/photos/autumn-leaf) - 免费高质量图片
  - [Pexels](https://www.pexels.com/search/autumn%20leaf/) - 免费图片
  - [Pixabay](https://pixabay.com/images/search/autumn%20leaf/) - 免费图片
  - [Flaticon](https://www.flaticon.com/search?word=autumn%20leaf) - 图标资源

- **搜索关键词**：
  - "autumn leaf"
  - "falling leaf"
  - "maple leaf"
  - "落叶"

- **图片要求**：
  - 格式：PNG（透明背景）
  - 尺寸：建议 32x32px 到 64x64px
  - 背景：透明
  - 数量：建议准备 3-5 种不同形状/颜色的叶子

## 2. 贴图放置位置

将下载的贴图放在以下目录：
```
src/assets/images/particles/
  - cherry-petal-1.png
  - cherry-petal-2.png
  - cherry-petal-3.png
  - leaf-1.png
  - leaf-2.png
  - leaf-3.png
```

## 3. 实现方案

### 方案 A：使用真实贴图（推荐）
- 使用 `Image` 对象加载 PNG 贴图
- 在 Canvas 上绘制图片而不是绘制形状
- 优点：更真实、更美观
- 缺点：需要准备贴图资源

### 方案 B：使用 SVG 路径绘制
- 使用 SVG 路径数据绘制更复杂的形状
- 优点：不需要外部资源
- 缺点：代码较复杂

### 方案 C：使用 Canvas 绘制更复杂的形状
- 改进当前的绘制函数，添加更多细节
- 优点：不需要外部资源
- 缺点：效果可能不如真实贴图

## 4. 推荐实现步骤

1. **下载贴图**：从推荐网站下载 3-5 种不同角度/形状的樱花花瓣和落叶
2. **处理贴图**：
   - 确保背景透明
   - 调整大小到合适尺寸（32-64px）
   - 使用图片编辑工具优化（可选）
3. **实现加载**：在代码中加载贴图并随机使用
4. **优化性能**：使用图片缓存，避免重复加载

