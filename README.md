# 测试文件收纳下载页

一个用于收纳、预览和下载测试文件的静态网页，支持按文件扩展名管理文档与图片，方便在测试不同文件类型上传、下载、预览能力时快速取用样例文件。

在线访问地址：
https://jessica-tech-dev.github.io/download_file/

## 功能特性

- 按扩展名收纳文档和图片文件。
- 展示文件名、文件类型、文件大小、更新时间和下载入口。
- 图片文件支持压缩预览图展示，浏览器可识别的图片类型会直接显示缩略预览。
- 支持新增上传文件或图片类型。
- 同类型文件重复上传时，会提示：`该类型文件已存在，是否需要覆盖`，并提供 `是` / `否` 选项。
- 使用浏览器 IndexedDB 本地存储文件，刷新页面后仍可保留已上传内容。
- 无需后端服务，直接通过 GitHub Pages 部署。

## 支持的文档类型

当前内置支持以下文档扩展名：

```text
doc、docx、ppt、pptx、pdf、xlsx、xls、txt、md
```

## 支持的图片类型

当前内置支持以下图片扩展名：

```text
jpg、jpeg、tif、wbmp、gif、HEIC、JPG、webp、dng、avci、bmp、heic、heif、ico、jpe、cgm、dwg、dxf
```

说明：页面会统一按小写扩展名识别文件类型，因此 `.HEIC`、`.JPG` 等大写扩展名也可以正常归类。

## 本地使用

直接打开 `index.html` 即可使用：

```text
index.html
```

也可以在项目目录启动本地静态服务：

```bash
python3 -m http.server 5173
```

然后访问：

```text
http://127.0.0.1:5173/index.html
```

## 部署方式

本项目是纯静态页面，已部署到 GitHub Pages。

如需重新部署或更新页面：

```bash
git add index.html README.md
git commit -m "Update file catalog page"
git push origin main
```

推送后 GitHub Pages 会自动更新。

## 文件存储说明

上传的文件不会提交到 GitHub 仓库，也不会上传到服务器。文件会保存在当前浏览器的 IndexedDB 中：

- 同一浏览器再次打开页面，可以继续看到之前上传过的文件。
- 更换浏览器、清理站点数据或使用其他设备时，需要重新上传文件。
- 如果上传大量大文件，可能受到浏览器本地存储空间限制。

## 项目结构

```text
.
├── index.html   # 页面主体、样式和交互逻辑
└── README.md    # 项目说明文档
```
