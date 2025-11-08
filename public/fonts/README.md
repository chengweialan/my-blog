# 字体文件说明

## 字体下载指南

### 推荐字体
- **方正行楷** (FZXingKai)
- **汉仪行楷** (HYXingKai)
- **华文行楷** (STXingkai)
- **叶根友行楷**

### 下载网站
1. **字客网**: https://www.fontke.com/
2. **字体天下**: https://www.fonts.net.cn/
3. **100font**: https://www.100font.com/

### 文件要求
- **格式**: 优先选择 `.woff2`（体积最小），其次 `.woff`，`.ttf` 也可用
- **大小**: 建议 2-5MB
- **文件名**: 下载后重命名为 `xingkai.woff2`、`xingkai.woff`、`xingkai.ttf`

### 放置位置
将字体文件放在此目录下：
```
public/fonts/
  ├── xingkai.woff2  (推荐)
  ├── xingkai.woff   (备选)
  └── xingkai.ttf    (备选)
```

### 使用说明
字体文件放置后，代码会自动加载。如果字体文件不存在，会使用系统默认的行草字体作为后备。

### 注意事项
- 确保字体文件有使用权限（商业字体需购买授权）
- 建议使用免费字体或已获得授权的字体
- 字体文件较大，建议使用 `woff2` 格式以减小体积

