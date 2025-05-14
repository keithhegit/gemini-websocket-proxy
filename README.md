# Gemini 2 Live API Demo (Cloudflare 安全代理版)

## 项目简介

本项目是一个纯前端的 Gemini 2.0 Flash Multimodal API 演示客户端，支持文本、音频、摄像头、屏幕输入与音频响应，适合部署在 Cloudflare Pages 等静态托管平台。后端 Gemini API 及 Deepgram 语音转写均通过 Cloudflare Worker 代理，保证密钥安全，前端无任何敏感信息泄露。

- 支持 Gemini 2.0 Flash 多模态实时交互
- 支持音频输入/输出、摄像头、屏幕分享
- 支持 Deepgram 语音转写（可选）
- 所有 API Key 均安全存储于 Cloudflare Worker Secret，前端无泄露风险
- 适合个人/团队安全部署

## 目录结构

```
/
  index.html
  /js
  /css
  /assets
  README.md
```

## 部署方法

### 1. Cloudflare Worker 代理 Gemini API

1. 注册 Cloudflare Workers，创建 Worker 项目。
2. 将 Gemini API Key 存为 Secret：
   ```bash
   npx wrangler secret put GEMINI_API_KEY
   ```
3. Worker 代码示例：
   ```js
   export default {
     async fetch(request, env) {
       // 代理 Gemini API
       const response = await fetch("https://geminiapi.keithhe2021.workers.dev/v1/gemini", {
         method: request.method,
         headers: {
           "Content-Type": "application/json",
           "Authorization": `Bearer ${env.GEMINI_API_KEY}`,
         },
         body: request.body,
       });
       return new Response(await response.text(), {
         status: response.status,
         headers: { "Content-Type": "application/json" },
       });
     },
   };
   ```
4. 部署后，获得你的 Gemini Worker 代理地址（如 `https://geminiapi.keithhe2021.workers.dev/v1/gemini`）。
5. 前端 `js/config/config.js` 已写死为你的 Worker 代理地址。

### 2. Cloudflare Worker 代理 Deepgram API

1. 创建 Deepgram Worker 项目。
2. 存储 Deepgram Key：
   ```bash
   npx wrangler secret put DEEPGRAM_API_KEY
   ```
3. Worker 代码示例：
   ```js
   export default {
     async fetch(request, env) {
       if (request.method !== "POST") {
         return new Response("Method Not Allowed", { status: 405 });
       }
       const response = await fetch("https://api.deepgram.com/v1/listen", {
         method: "POST",
         headers: {
           "Authorization": `Token ${env.DEEPGRAM_API_KEY}`,
           "Content-Type": request.headers.get("Content-Type"),
         },
         body: request.body,
       });
       return new Response(response.body, response);
     }
   };
   ```
4. 部署后，获得你的 Deepgram Worker 代理地址（如 `https://deepgram-proxy.xxx.workers.dev/v1/listen`）。
5. 前端 `js/config/config.js` 已写死为你的 Worker 代理地址。

### 3. 部署前端到 Cloudflare Pages

1. 推送本项目代码到 GitHub。
2. 在 Cloudflare Pages 新建项目，选择你的仓库。
3. Build command 留空，output directory 设为 `.`（根目录）。
4. 部署完成后即可访问。

### 4. 本地开发预览

```bash
python3 -m http.server 8000
# 或 npx http-server 8000
# 访问 http://localhost:8000
```

## 注意事项
- 前端无任何 API Key 设置项，所有密钥均安全存储于 Worker Secret。
- 如需更换代理地址，请修改 `js/config/config.js`。
- 如需自定义功能，请参考各模块源码。

## 参考
- [Cloudflare Pages 官方文档](https://developers.cloudflare.com/pages/)
- [Cloudflare Workers 官方文档](https://developers.cloudflare.com/workers/)
- [Gemini API 官方文档](https://ai.google.dev/)
- [Deepgram API 官方文档](https://developers.deepgram.com/)
