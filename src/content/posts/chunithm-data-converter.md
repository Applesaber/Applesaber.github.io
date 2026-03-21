---
title: '国服中二节奏成绩导入MuNET'
published: 2026-03-22T00:00:00+08:00
description: '将落雪查分器 / 水鱼查分器的 Chunithm 成绩数据转换为 MuNET 可导入的 JSON 格式的开源工具。'
category: '项目'
tags: [Tool, MuNET, 开源]
author: Applesaber
sourceLink: 'https://github.com/MuNET-OSS/chunithm-data-converter'
---

::github{repo="MuNET-OSS/chunithm-data-converter"}

:::important
该转换器功能现已**原生集成至 MuNET 中**！可以在MuNET的 [中二节奏数据导出/导入页面](https://portal.mumur.net/chu3/transfer) 中使用
:::

将落雪查分器 (Lxns) 或 水鱼查分器 (Diving-Fish) 的 Chunithm (中二节奏) 成绩数据，转换为 **MuNET** 兼容的 JSON 导入格式。

目前该工具已经支持三种使用方式：**基于 Vue 的纯前端网页**、**基于 Flask 的纯净本地 Web 接口**，以及**Python 命令行 CLI**。

**快速体验**：[https://www.applesaber.site/chunithm-data-converter/](https://www.applesaber.site/chunithm-data-converter/)

---

### 特点
- **部署方式多**：提供 Python Web 端、Python 命令行端以及 workflow 部署版
- **模式多**：深度适配落雪查分器个人/开发者模式，水鱼查分器个人/开发者模式，同时支持从 CSV 离线导入
- **更好看的UI**：前端风格为 "简约现代亚克力" 风格，配合 Naive UI 组件库 （使用Gemini 3.1 Pro 润化）
- **解决跨域问题**：针对水鱼 API 严格的 CORS 安全策略，workflow 内置了 Cloudflare Worker 代理节点；本地部署版则通过 Python 直接向 API 发起请求

---

## 使用指南与数据源支持

:::note
工具**不会**保存、上传你的任何成绩与凭证信息。
所有的解析和请求都是**纯本地**或**受信任的环境**中完成的，尽可放心使用。
:::

| 模式 | 适用场景 | 认证凭证 |
|---|---|---|
| **落雪个人** | 获取自己的成绩 | 个人访问 Token |
| **落雪开发者** | 获取任意好友成绩 | 开发者 Token + 对方的好友码 |
| **水鱼个人** | 获取自己的成绩 | 水鱼 Import-Token |
| **水鱼开发者** | 获取特定玩家成绩 | Developer-Token + 对方用户名 |
| **CSV 上传** | API 死了或只想离线转换 | 落雪/水鱼导出的 CSV 文件 |

---

## GH-Pages 部署版

最简单的方式，直接打开网站，无需环境配置。
页面由 Vue 3 + TypeScript + Vite 构建。

1. 进入网页，选择你拥有的数据源
2. 填入对应的 Token（开发者令牌已在系统后台静默配置，无需用户填写）
3. 点击“开始转换”，随后下载 JSON 文件。
4. 将文件导入至 MuNET。

:::tip
**前端编译说明（开发者）**

推荐使用 `npx` 运行 vite 编译指令：
```bash
cd web
npm install
npx vite build
```
:::

---

## Python 命令行版

```bash
# 安装依赖
pip install -r requirements.txt

# 1. API 模式 - 水鱼个人查分
python main.py api --mode shuiyu --shuiyu-import-token 你的Token -o my_data.json

# 2. CSV 本地文件模式
python main.py csv --input scores.csv --username 玩家名
```

---

## 本地 Web 接口 (web.py)

### web.py 和 workflow版 有什么区别
该方案将 `chunithm_api_converter` 与 `convert_chunithm_scores` 这两个核心 Python 模块直接导入
* **没有 CORS 影响**：Python 后端作为独立服务去请求第三方 API，根本不受浏览器跨域安全策略影响

### 部署方法
```bash
# 1. 编译前端界面 (Web 目录下的静态资源)
cd web
npm install
npx vite build
cd ..

# 2. 安装 Python 核心及 Web 依赖
pip install -r requirements.txt

# 3. 设定您的环境私钥 （非必须项）
export LXNS_DEVELOPER_TOKEN="你的开发者落雪Token"
export SHUIYU_DEVELOPER_TOKEN="你的开发者水鱼Token"
export PORT="5000"

# 4. 启动！
python web.py
```

:::important
**关于环境变量与安全**

- `LXNS_DEVELOPER_TOKEN` 和 `SHUIYU_DEVELOPER_TOKEN`：**千万不要**将这些开发者 Token 暴露给用户或者写死在公开代码里。正确做法是在启动 `web.py` 前通过终端的 `export` 或根目录配置 `.env` 文件来静默注入，后端在处理请求时会自动从服务器环境中读取这些凭证，以此保障你的开发者安全。
- 对于本文提到的 `GH-Pages 部署版`，此机制则是通过 GitHub Actions 的 Repository Secrets 功能实现的，部署流水线会在编译时将这些 Token 隐藏起来。
:::

现在访问 `http://localhost:5000` ，你就能看到页面了 同时它还提供了对外的 POST API：
- `POST /api/convert` 
- `POST /api/convert/csv`

如果觉得帮助到你记得给我项目点个 star⭐ 谢谢喵
