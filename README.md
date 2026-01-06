# cfwork-to-python
cfwork无损让AI大模型转为python本地版本，绕过cfwork子请求50次限制等等一日十万次套餐请求限制
```
我将提供一个完整的 cfwork的完整 项目源代码（你务必自动识别他的架构和他的目的以及他主要是为了什么等等操作）。你的任务是：
核心转换： 将该 cfwork项目的后端代理逻辑，完整、无损地迁移到一个高性能的 python项目中。
我可以给你几个例子你可以查看一下，方便你在转换过程中还可以进行优化等等：
你看这是原cfwork版本示例项目：
/**
 * =================================================================================
 * 项目: aiimagetoimage-2api (Cloudflare Worker 终极行为复刻版)
 * 版本: 1.7.0 (代号: Chimera Reborn - Absolute Stealth)
 * 修复: 1. 彻底修复 429 识别问题 2. 修复 JSON 解析错误 3. 恢复全功能驾驶舱 UI
 * =================================================================================
 */

const CONFIG = {
  PROJECT_NAME: "aiimagetoimage-2api",
  PROJECT_VERSION: "1.7.0",
  API_MASTER_KEY: "1", 

  // 上游地址
  UPSTREAM_ORIGIN: "https://aiimagetoimage.io",
  GENERATE_ENDPOINT: "https://api.aiimagetoimage.io/api/img2img/image-generate/image2image",
  STATUS_ENDPOINT: "https://api.aiimagetoimage.io/api/result/get",
  ASSETS_PRELOAD_URL: "https://aiimagetoimage.io/assets/image/home/demo3.png",
  GA_ENDPOINT: "https://region1.google-analytics.com/g/collect",

  // 模型配置
  MODELS: [
    { id: "nano_banana", name: "Nano Banana (快速/推荐)" },
    { id: "standard", name: "Standard (标准)" }
  ],
  DEFAULT_MODEL: "nano_banana",
  ASPECT_RATIOS: ["match_input_image", "1:1", "3:2", "2:3", "9:16", "16:9", "3:4", "4:3"],
  POLLING_TIMEOUT: 300000, 
};

// --- [第一部分: 身份与指纹伪装引擎] ---

class IdentityManager {
  /**
   * 生成随机浏览器指纹 (模拟无痕模式)
   */
  static createIdentity() {
    // 随机化 Chrome 小版本号，模拟不同用户
    const chromeVersion = `143.0.${Math.floor(Math.random() * 9999)}.${Math.floor(Math.random() * 999)}`;
    
    return {
      headers: {
        "accept": "*/*",
        "accept-encoding": "gzip, deflate, br, zstd",
        "accept-language": "zh-CN,zh;q=0.9",
        "origin": "https://aiimagetoimage.io",
        "priority": "u=1, i",
        "referer": "https://aiimagetoimage.io/",
        "sec-ch-ua": `"Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"`,
        "sec-ch-ua-mobile": "?0",
        "sec-ch-ua-platform": '"Windows"',
        "sec-fetch-dest": "empty",
        "sec-fetch-mode": "cors",
        "sec-fetch-site": "same-site",
        "user-agent": `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/${chromeVersion} Safari/537.36`
        // 注意：绝对不发送 X-Forwarded-For，避免暴露代理身份
      }
    };
  }

  /**
   * 模拟 Google Analytics 行为
   */
  static async simulateGA(identity) {
    const cid = `${Math.floor(Math.random() * 1000000000)}.${Math.floor(Date.now() / 1000)}`;
    const params = new URLSearchParams({
      v: "2",
      tid: "G-QN0ECG686N",
      gtm: "45je5ca1v9229895114za200zd9229895114",
      _p: Date.now().toString(),
      cid: cid,
      ul: "zh-cn",
      sr: "1920x1080",
      en: "page_view",
      dl: "https://aiimagetoimage.io/",
      dt: "FREE AI Image to Image Generator: Pro Edits via Text Prompt"
    });

    try {
      await fetch(`${CONFIG.GA_ENDPOINT}?${params.toString()}`, {
        method: "POST",
        headers: {
          ...identity.headers,
          "sec-fetch-mode": "no-cors",
          "sec-fetch-site": "cross-site"
        }
      });
    } catch (e) {}
  }

  /**
   * 模拟静态资源预加载
   */
  static async preload(identity) {
    try {
      await fetch(CONFIG.ASSETS_PRELOAD_URL, {
        method: "GET",
        headers: {
          ...identity.headers,
          "sec-fetch-dest": "image",
          "sec-fetch-mode": "no-cors"
        }
      });
    } catch (e) {}
  }
}

// --- [第二部分: 核心业务逻辑] ---

async function submitTaskWithSimulation(prompt, imageBlob, model, ratio, logCallback) {
  const identity = IdentityManager.createIdentity();
  
  await logCallback("DEBUG", `>>> [Identity] 模拟全新无痕浏览器指纹已就绪`);
  
  // 行为模拟 1: 访问首页并加载资源
  await logCallback("DEBUG", `>>> [Handshake] 模拟首页访问与资源预加载...`);
  await IdentityManager.preload(identity);
  
  // 行为模拟 2: 发送 GA 统计 (关键：让上游认为你是真实访客)
  await logCallback("DEBUG", `>>> [Handshake] 模拟 Google Analytics 埋点上报...`);
  await IdentityManager.simulateGA(identity);
  
  // 模拟人类操作延迟
  await new Promise(r => setTimeout(r, 1500));

  // 构造 Multipart 请求
  const formData = new FormData();
  if (imageBlob) {
    const finalBlob = new Blob([await imageBlob.arrayBuffer()], { type: "image/jpeg" });
    formData.append("image", finalBlob, "产品1.jpg");
    await logCallback("DEBUG", `>>> [Payload] 图片已封装 (Size: ${finalBlob.size} bytes)`);
  }
  
  formData.append("prompt", prompt || "High quality");
  formData.append("negative_prompt", "");
  formData.append("model_type", model || CONFIG.DEFAULT_MODEL);
  formData.append("aspect_ratio", ratio || "match_input_image");

  await logCallback("DEBUG", `>>> [UPSTREAM_REQUEST] 正在提交任务到上游接口...`);

  const response = await fetch(CONFIG.GENERATE_ENDPOINT, {
    method: "POST",
    headers: identity.headers,
    body: formData
  });

  const responseText = await response.text();
  await logCallback("DEBUG", `<<< [UPSTREAM_RESPONSE] Status: ${response.status}`);
  await logCallback("DEBUG", `<<< [UPSTREAM_RESPONSE] Body: ${responseText}`);

  let data;
  try {
    data = JSON.parse(responseText);
  } catch (e) {
    throw new Error(`上游响应解析失败: ${responseText.substring(0, 100)}`);
  }

  if (data.code !== 200) {
    if (data.code === 429) {
      throw new Error("上游触发 429 限制。原因：Cloudflare 节点 IP 已达今日上限。请尝试更换 Worker 区域或稍后再试。");
    }
    throw new Error(`上游业务错误: ${JSON.stringify(data.message)}`);
  }

  return { jobId: data.result.job_id, identity };
}

// --- [第三部分: Worker 路由与接口适配] ---

export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    const apiKey = env.API_MASTER_KEY || CONFIG.API_MASTER_KEY;

    // 处理跨域
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        status: 204,
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
          "Access-Control-Allow-Headers": "*"
        }
      });
    }

    // 首页 UI
    if (url.pathname === '/') return handleUI(request, apiKey);
    
    // 鉴权
    const auth = request.headers.get("Authorization");
    if (apiKey !== "1" && auth !== `Bearer ${apiKey}`) {
      return new Response(JSON.stringify({ error: "Unauthorized" }), { status: 401 });
    }

    // 模型列表
    if (url.pathname === '/v1/models') {
      return new Response(JSON.stringify({
        object: "list",
        data: CONFIG.MODELS.map(m => ({ id: m.id, object: "model", created: Date.now(), owned_by: "aiimagetoimage" }))
      }), { headers: { "Content-Type": "application/json", "Access-Control-Allow-Origin": "*" } });
    }

    // Chat 接口 (适配 Vision)
    if (url.pathname === '/v1/chat/completions') return handleChat(request, ctx);
    
    // 状态查询代理
    if (url.pathname === '/v1/query/status') return handleStatusProxy(request);

    return new Response("Not Found", { status: 404 });
  }
};

async function handleStatusProxy(request) {
  const jobId = new URL(request.url).searchParams.get("job_id");
  const identity = IdentityManager.createIdentity();
  const response = await fetch(`${CONFIG.STATUS_ENDPOINT}?job_id=${jobId}`, {
    headers: identity.headers
  });
  return new Response(response.body, { headers: { "Content-Type": "application/json", "Access-Control-Allow-Origin": "*" } });
}

async function handleChat(request, ctx) {
  const body = await request.json();
  const messages = body.messages || [];
  const lastMsg = messages[messages.length - 1];
  const isWebUI = body.is_web_ui === true;

  let prompt = "";
  let imageBlob = null;

  // 解析多模态内容
  if (Array.isArray(lastMsg.content)) {
    for (const part of lastMsg.content) {
      if (part.type === 'text') prompt += part.text;
      if (part.type === 'image_url') {
        const res = await fetch(part.image_url.url);
        imageBlob = await res.blob();
      }
    }
  } else {
    prompt = lastMsg.content;
  }

  const { readable, writable } = new TransformStream();
  const writer = writable.getWriter();
  const encoder = new TextEncoder();

  const logToClient = async (tag, msg) => {
    const data = { debug_log: { tag, msg } };
    await writer.write(encoder.encode(`data: ${JSON.stringify(data)}\n\n`));
  };

  ctx.waitUntil((async () => {
    try {
      const { jobId, identity } = await submitTaskWithSimulation(
        prompt, 
        imageBlob, 
        body.model, 
        body.aspect_ratio, 
        logToClient
      );

      if (isWebUI) {
        // Web 模式：直接返回 JobID 让前端轮询
        await writer.write(encoder.encode(`data: ${JSON.stringify({ job_id: jobId, status: "submitted" })}\n\n`));
      } else {
        // API 模式：Worker 内部轮询
        let completed = false;
        let startTime = Date.now();
        while (!completed && Date.now() - startTime < CONFIG.POLLING_TIMEOUT) {
          const statusRes = await fetch(`${CONFIG.STATUS_ENDPOINT}?job_id=${jobId}`, { headers: identity.headers });
          const statusData = await statusRes.json();
          if (statusData.code === 200 && statusData.result?.image_url) {
            const url = statusData.result.image_url[0];
            const chunk = { 
              id: `chatcmpl-${crypto.randomUUID()}`, 
              object: "chat.completion.chunk", 
              choices: [{ delta: { content: `![Generated Image](\${url})` }, finish_reason: "stop" }] 
            };
            await writer.write(encoder.encode(`data: ${JSON.stringify(chunk)}\n\n`));
            completed = true;
          } else {
            await new Promise(r => setTimeout(r, 3000));
          }
        }
      }
    } catch (e) {
      await logToClient("ERROR", e.message);
      await writer.write(encoder.encode(`data: ${JSON.stringify({ error: { message: e.message } })}\n\n`));
    } finally {
      await writer.write(encoder.encode("data: [DONE]\n\n"));
      await writer.close();
    }
  })());

  return new Response(readable, { headers: { "Content-Type": "text/event-stream", "Access-Control-Allow-Origin": "*" } });
}

// --- [第四部分: 开发者驾驶舱 UI (全功能版)] ---

function handleUI(request, apiKey) {
  const origin = new URL(request.url).origin;
  const html = `<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>${CONFIG.PROJECT_NAME} - 终极驾驶舱</title>
    <style>
        :root {
            --bg: #0D0D0D; --panel: #161616; --border: #262626; --text: #E5E5E5;
            --primary: #FFBF00; --success: #4ADE80; --error: #F87171;
        }
        body { font-family: 'Inter', system-ui, sans-serif; background: var(--bg); color: var(--text); margin: 0; height: 100vh; display: flex; overflow: hidden; }
        .sidebar { width: 400px; background: var(--panel); border-right: 1px solid var(--border); padding: 24px; display: flex; flex-direction: column; overflow-y: auto; }
        .main { flex: 1; display: flex; flex-direction: column; padding: 24px; background: #000; }
        
        .card { background: #1F1F1F; padding: 16px; border-radius: 12px; border: 1px solid var(--border); margin-bottom: 16px; }
        .label { font-size: 11px; color: #737373; margin-bottom: 8px; display: block; font-weight: bold; text-transform: uppercase; }
        .code-block { font-family: monospace; font-size: 12px; color: var(--primary); background: #000; padding: 10px; border-radius: 6px; word-break: break-all; border: 1px solid #333; }
        
        input, select, textarea { width: 100%; background: #262626; border: 1px solid #333; color: #fff; padding: 10px; border-radius: 6px; margin-bottom: 10px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background: var(--primary); border: none; border-radius: 6px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        button:hover { filter: brightness(1.1); }
        button:disabled { background: #444; cursor: not-allowed; }

        .upload-area { border: 2px dashed #444; border-radius: 8px; padding: 20px; text-align: center; cursor: pointer; margin-bottom: 10px; position: relative; }
        #preview { max-width: 100%; max-height: 200px; display: none; margin: 0 auto; border-radius: 4px; }

        .terminal { flex: 1; background: #050505; border: 1px solid var(--border); border-radius: 12px; display: flex; flex-direction: column; overflow: hidden; }
        .terminal-header { background: #1A1A1A; padding: 10px 20px; border-bottom: 1px solid var(--border); font-size: 12px; display: flex; justify-content: space-between; }
        .output { flex: 1; padding: 20px; overflow-y: auto; font-family: monospace; font-size: 13px; line-height: 1.6; }
        .log-item { margin-bottom: 6px; border-left: 2px solid #333; padding-left: 10px; }
        .log-DEBUG { color: #00FF41; }
        .log-ERROR { color: var(--error); }
        
        .progress-container { height: 4px; background: #222; width: 100%; }
        .progress-bar { height: 100%; background: var(--primary); width: 0%; transition: 0.3s; }
        
        .result-img { max-width: 100%; border-radius: 8px; border: 1px solid var(--primary); margin-top: 10px; }
    </style>
</head>
<body>
    <div class="sidebar">
        <h2 style="color:var(--primary); margin-top:0;">🖼️ AI Cockpit <small>v${CONFIG.PROJECT_VERSION}</small></h2>
        
        <div class="card">
            <span class="label">API KEY</span>
            <div class="code-block">${apiKey}</div>
        </div>

        <div class="card">
            <span class="label">配置参数</span>
            <select id="model">
                ${CONFIG.MODELS.map(m => `<option value="${m.id}">${m.name}</option>`).join('')}
            </select>
            <select id="ratio">
                ${CONFIG.ASPECT_RATIOS.map(r => `<option value="${r}">${r}</option>`).join('')}
            </select>
            
            <div class="upload-area" onclick="document.getElementById('fileInput').click()">
                <div id="upload-text">点击上传参考图</div>
                <img id="preview">
                <input type="file" id="fileInput" hidden accept="image/*">
            </div>

            <textarea id="prompt" rows="3" placeholder="输入提示词..."></textarea>
            <button id="genBtn">🚀 开始生成 (全链路模拟)</button>
        </div>
    </div>

    <div class="main">
        <div class="terminal">
            <div class="terminal-header">
                <span>TERMINAL OUTPUT</span>
                <span id="status">READY</span>
            </div>
            <div class="output" id="output">
                <div style="color:#555">等待任务提交...</div>
            </div>
            <div class="progress-container"><div class="progress-bar" id="pb"></div></div>
        </div>
    </div>

    <script>
        const API_KEY = "${apiKey}";
        let selectedBlob = null;

        // 图片预览
        document.getElementById('fileInput').onchange = e => {
            const file = e.target.files[0];
            if (file) {
                selectedBlob = file;
                const reader = new FileReader();
                reader.onload = e => {
                    document.getElementById('preview').src = e.target.result;
                    document.getElementById('preview').style.display = 'block';
                    document.getElementById('upload-text').style.display = 'none';
                };
                reader.readAsDataURL(file);
            }
        };

        function addLog(tag, msg) {
            const out = document.getElementById('output');
            const div = document.createElement('div');
            div.className = 'log-item log-' + tag;
            div.innerHTML = \`[\${new Date().toLocaleTimeString()}] [\${tag}] \${msg}\`;
            out.appendChild(div);
            out.scrollTop = out.scrollHeight;
        }

        async function run() {
            const btn = document.getElementById('genBtn');
            const pb = document.getElementById('pb');
            const status = document.getElementById('status');
            const prompt = document.getElementById('prompt').value;

            if (!prompt) return alert("请输入提示词");

            btn.disabled = true;
            document.getElementById('output').innerHTML = '';
            pb.style.width = '10%';
            status.innerText = 'SIMULATING...';

            try {
                let content = [{ type: "text", text: prompt }];
                if (selectedBlob) {
                    const base64 = await new Promise(r => {
                        const reader = new FileReader();
                        reader.onload = () => r(reader.result);
                        reader.readAsDataURL(selectedBlob);
                    });
                    content.push({ type: "image_url", image_url: { url: base64 } });
                }

                const res = await fetch('/v1/chat/completions', {
                    method: 'POST',
                    headers: { 'Authorization': 'Bearer ' + API_KEY, 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        model: document.getElementById('model').value,
                        aspect_ratio: document.getElementById('ratio').value,
                        messages: [{ role: 'user', content: content }],
                        is_web_ui: true
                    })
                });

                const reader = res.body.getReader();
                const decoder = new TextDecoder();
                let jobId = null;

                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    const chunk = decoder.decode(value);
                    const lines = chunk.split('\\n');
                    
                    for (let line of lines) {
                        if (!line.trim() || line === 'data: [DONE]') continue; // 修复解析错误的关键
                        
                        if (line.startsWith('data: ')) {
                            try {
                                const data = JSON.parse(line.substring(6));
                                if (data.debug_log) addLog(data.debug_log.tag, data.debug_log.msg);
                                if (data.job_id) jobId = data.job_id;
                                if (data.error) throw new Error(data.error.message);
                            } catch (e) {
                                // 忽略非 JSON 行
                            }
                        }
                    }
                }

                if (!jobId) throw new Error("未能获取 JobID，请检查日志。");

                status.innerText = 'POLLING...';
                pb.style.width = '50%';

                // 轮询结果
                while (true) {
                    const poll = await fetch(\`/v1/query/status?job_id=\${jobId}\`, {
                        headers: { 'Authorization': 'Bearer ' + API_KEY }
                    });
                    const pollData = await poll.json();
                    
                    if (pollData.code === 200 && pollData.result?.image_url) {
                        const url = pollData.result.image_url[0];
                        addLog("SUCCESS", "生成成功！");
                        document.getElementById('output').innerHTML += \`<br><img src="\${url}" class="result-img"><br><a href="\${url}" target="_blank" style="color:var(--primary)">点击下载原图</a>\`;
                        pb.style.width = '100%';
                        status.innerText = 'COMPLETED';
                        break;
                    } else if (pollData.code === 202) {
                        addLog("DEBUG", "任务处理中...");
                        pb.style.width = (parseInt(pb.style.width) + 5) + '%';
                    } else {
                        throw new Error("轮询异常: " + JSON.stringify(pollData));
                    }
                    await new Promise(r => setTimeout(r, 3000));
                }

            } catch (e) {
                addLog("ERROR", e.message);
                status.innerText = 'FAILED';
                pb.style.width = '0%';
            } finally {
                btn.disabled = false;
            }
        }

        document.getElementById('genBtn').onclick = run;
    </script>
</body>
</html>`;
  return new Response(html, { headers: { "Content-Type": "text/html;charset=UTF-8" } });
}


然后接下来这是转成的python本地版本，操作简单，就一个main，然后呢况且无损并且功能完全具备，并且总体来说软件化的方便快速启动后期打包等等的，并且如果测试期间你可以编写一个bat脚本，bat脚本我后面也会提供给你，这个bat脚本为了方便就是让用户可以快速启动，方便依赖自动下载补全等等以及虚拟环境等等还有就是内嵌python等等一系列的支持，让小白也能轻松玩转

# -*- coding: utf-8 -*-
import os
import json
import time
import uuid
import threading
import random
import logging
import base64
import webview  # 核心：原生窗口容器
from datetime import datetime
from flask import Flask, request, jsonify, Response, stream_with_context
from flask_cors import CORS
import requests

# =================================================================
# 核心配置与统计管理 (Configuration & Stats)
# =================================================================
CONFIG = {
    "PROJECT_NAME": "AI 图像驾驶舱 终极版",
    "VERSION": "5.0.0",
    "PORT": 5896,
    "API_KEY": "1",
    "UPSTREAM": "https://api.aiimagetoimage.io",
    "UPSTREAM_BACKUP": [],  # 备用API地址列表，如 ["https://backup-api.example.com"]
    "GA_URL": "https://region1.google-analytics.com/g/collect",
    "DATA_FILE": "cockpit_pro_data.json",
    "USER_AGENT": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36",
    # API类型配置: "default", "cherry", "openai", "lmstudio", "ollama"
    "API_TYPE": "default",
    # Cherry Studio等本地API地址
    "CHERRY_STUDIO_URL": "http://127.0.0.1:8080",
    "OPENAI_BASE_URL": "http://127.0.0.1:1234/v1",
    # 模型映射：前端显示名称 -> 实际API模型标识
    "MODEL_MAPPING": {
        "nano_banana": "nano_banana",
        "standard": "standard"
    },
    # 可用模型列表（用于/v1/models端点）
    "MODELS": [
        {"id": "nano_banana", "name": "Nano Banana (极速)", "supports_images": True},
        {"id": "standard", "name": "Standard (高清)", "supports_images": True}
    ]
}

app = Flask(__name__)
CORS(app)
logging.getLogger('werkzeug').setLevel(logging.ERROR)

# 数据持久化逻辑
def get_default_data():
    return {
        "stats": {
            "total_calls": 0,
            "success_calls": 0,
            "failed_calls": 0,
            "last_call_time": "无记录"
        },
        "history": [],
        "settings": {"theme": "obsidian"}
    }

def load_data():
    if os.path.exists(CONFIG["DATA_FILE"]):
        try:
            with open(CONFIG["DATA_FILE"], "r", encoding="utf-8") as f:
                return json.load(f)
        except:
            return get_default_data()
    return get_default_data()

def save_data(data):
    with open(CONFIG["DATA_FILE"], "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

# =================================================================
# 核心引擎 (Engine)
# =================================================================
class ImageEngine:
    @staticmethod
    def get_headers():
        return {
            "accept": "*/*",
            "accept-language": "zh-CN,zh;q=0.9",
            "origin": "https://aiimagetoimage.io",
            "referer": "https://aiimagetoimage.io/",
            "sec-ch-ua": '"Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"',
            "sec-ch-ua-mobile": "?0",
            "sec-ch-ua-platform": '"Windows"',
            "sec-fetch-dest": "empty",
            "sec-fetch-mode": "cors",
            "sec-fetch-site": "same-site",
            "user-agent": CONFIG["USER_AGENT"]
        }

    @staticmethod
    def simulate_ga():
        cid = f"{random.randint(1000000000, 9999999999)}.{int(time.time())}"
        params = {"v": "2", "tid": "G-QN0ECG686N", "cid": cid, "en": "page_view", "dl": "https://aiimagetoimage.io/"}
        try: requests.post(CONFIG["GA_URL"], params=params, timeout=5)
        except: pass

    @staticmethod
    def get_api_url(model_id):
        """根据模型ID和API类型返回对应的API地址"""
        api_type = CONFIG.get("API_TYPE", "default")
        
        # 获取实际模型标识（通过映射）
        actual_model = CONFIG["MODEL_MAPPING"].get(model_id, model_id)
        
        if api_type == "cherry":
            # Cherry Studio API (假设兼容OpenAI格式)
            return f"{CONFIG['CHERRY_STUDIO_URL']}/v1/chat/completions"
        elif api_type == "openai":
            # OpenAI兼容API (LM Studio, Ollama等)
            return f"{CONFIG['OPENAI_BASE_URL']}/chat/completions"
        else:
            # 默认API
            return f"{CONFIG['UPSTREAM']}/api/img2img/image-generate/image2image"

    @staticmethod
    def get_all_api_urls(model_id):
        """获取所有可用的API地址（主API + 备用API）"""
        api_type = CONFIG.get("API_TYPE", "default")
        
        if api_type == "default":
            urls = [f"{CONFIG['UPSTREAM']}/api/img2img/image-generate/image2image"]
            # 添加备用API地址
            for backup in CONFIG.get("UPSTREAM_BACKUP", []):
                urls.append(f"{backup}/api/img2img/image-generate/image2image")
            return urls
        else:
            # 其他API类型返回单个地址
            return [ImageEngine.get_api_url(model_id)]

    @staticmethod
    def prepare_request_data(api_type, model_id, prompt, image_data=None, aspect_ratio="match_input_image"):
        """根据不同API类型准备请求数据"""
        actual_model = CONFIG["MODEL_MAPPING"].get(model_id, model_id)
        
        if api_type in ["cherry", "openai"]:
            # OpenAI兼容格式
            messages = [{"role": "user", "content": []}]
            if prompt:
                messages[0]["content"].append({"type": "text", "text": prompt})
            if image_data and "base64," in image_data:
                messages[0]["content"].append({
                    "type": "image_url",
                    "image_url": {"url": image_data}
                })
            
            return {
                "model": actual_model,
                "messages": messages,
                "stream": True
            }
        else:
            # 默认API格式
            return {
                "prompt": prompt,
                "negative_prompt": "",
                "model_type": actual_model,
                "aspect_ratio": aspect_ratio
            }

    @staticmethod
    def get_api_headers(api_type):
        """根据API类型获取请求头"""
        if api_type in ["cherry", "openai"]:
            # OpenAI兼容API头
            return {
                "Content-Type": "application/json",
                "Authorization": f"Bearer {CONFIG['API_KEY']}"
            }
        else:
            # 默认API头
            return ImageEngine.get_headers()

    @staticmethod
    def process_api_response(api_type, response):
        """处理不同API类型的响应"""
        if api_type in ["cherry", "openai"]:
            # OpenAI兼容API响应
            return response.json()
        else:
            # 默认API响应
            return response.json()

# =================================================================
# API 路由 (Routes)
# =================================================================

@app.route('/api/data', methods=['GET'])
def get_all_data():
    return jsonify(load_data())

@app.route('/api/theme', methods=['POST'])
def set_theme():
    theme = request.json.get("theme")
    data = load_data()
    data["settings"]["theme"] = theme
    save_data(data)
    return jsonify({"status": "success"})

@app.route('/v1/models', methods=['GET'])
def list_models():
    """返回OpenAI兼容的模型列表，包含显示名称和图像支持信息"""
    models = []
    for model in CONFIG["MODELS"]:
        models.append({
            "id": model["id"],
            "object": "model",
            "created": int(time.time()),
            "owned_by": "system",
            "permission": [],
            "root": model["id"],
            "parent": None,
            # 扩展字段，用于前端显示
            "display_name": model.get("name", model["id"]),
            "supports_images": model.get("supports_images", True)
        })
    return jsonify({
        "object": "list",
        "data": models
    })

@app.route('/v1/chat/completions', methods=['POST'])
def chat_completions():
    body = request.json
    messages = body.get("messages", [])
    last_msg = messages[-1]["content"]
    
    prompt = ""
    image_data = None

    if isinstance(last_msg, list):
        for part in last_msg:
            if part["type"] == "text": prompt = part["text"]
            if part["type"] == "image_url": image_data = part["image_url"]["url"]
    else:
        prompt = last_msg

    def generate():
        # 辅助函数：生成符合OpenAI规范的调试信息Chunk
        def debug_chunk(msg):
             chunk = {
                "id": f"chatcmpl-{uuid.uuid4()}",
                "object": "chat.completion.chunk",
                "created": int(time.time()),
                "model": body.get("model", "nano_banana"),
                "choices": [{
                    "index": 0, 
                    # 将调试日志做为内容输出，或者使用特殊的注释格式让前端处理
                    # 这里为了兼容性，我们直接输出为文本，但加上特定的前缀
                    "delta": {"content": f"\n`{msg}`\n"}, 
                    "finish_reason": None
                }]
            }
             return f"data: {json.dumps(chunk)}\n\n"

        yield debug_chunk(">>> [系统] 正在初始化原生渲染引擎...")
        ImageEngine.simulate_ga()
        
        files = {}
        if image_data and "base64," in image_data:
            try:
                header, encoded = image_data.split(",", 1)
                img_bytes = base64.b64decode(encoded)
                files['image'] = ('product.jpg', img_bytes, 'image/jpeg')
            except:
                yield debug_chunk(">>> [错误] 图像解码失败")
        else:
             # 如果没有提供图片，使用默认的1x1黑色像素图片以满足API必须有图片的要求
             try:
                 # 1x1 黑色 JPEG 像素
                 pixel_b64 = "/9j/4AAQSkZJRgABAQEASABIAAD/2wBDAP//////////////////////////////////////////////////////////////////////////////////////wgALCAABAAEBAREA/8QAFBABAAAAAAAAAAAAAAAAAAAAAP/aAAgBAQABPxA="
                 img_bytes = base64.b64decode(pixel_b64)
                 files['image'] = ('pixel.jpg', img_bytes, 'image/jpeg')
                 yield debug_chunk(">>> [提示] 未提供参考图，已自动填充空白底图")
             except:
                 pass

        # 准备请求数据和URL
        
        # 准备请求数据（使用上游API格式）
        data = {
            "prompt": prompt,
            "negative_prompt": "",
            "model_type": body.get("model", "nano_banana"),
            "aspect_ratio": body.get("aspect_ratio", "match_input_image")
        }
        
        # 伪造IP (Soft IP Spoofing)
        def get_random_ip():
            return f"{random.randint(1,255)}.{random.randint(0,255)}.{random.randint(0,255)}.{random.randint(0,255)}"
            
        spoofed_headers = ImageEngine.get_headers()
        fake_ip = get_random_ip()
        spoofed_headers["X-Forwarded-For"] = fake_ip
        spoofed_headers["X-Real-IP"] = fake_ip
        
        # 代理配置
        # 【重要】如果上游封锁了IP，必须使用真实代理（梯子）。
        # 这里默认尝试连接常见的本地代理端口 7890 (Clash/v2ray等)。
        # 如果你的代理端口不同（如 10809），请修改下面的端口号。
        # 如果没有代理，请将 proxies 设置为 None
        proxies = {
            "http": "http://127.0.0.1:7890",
            "https": "http://127.0.0.1:7890"
        }
        
        try:
            yield debug_chunk(">>> [网络] 正在通过代理隧道连接上游 (Proxy: 127.0.0.1:7890)...")
            
            # 直接请求上游API
            resp = requests.post(
                f"{CONFIG['UPSTREAM']}/api/img2img/image-generate/image2image",
                headers=spoofed_headers,
                data=data,
                files=files,
                timeout=30,
                proxies=proxies
            )
            res_json = resp.json()
            
            if res_json.get("code") == 200:
                job_id = res_json["result"]["job_id"]
                yield debug_chunk(f">>> [成功] 任务已进入渲染队列: {job_id}")
                
                # 轮询结果
                start_time = time.time()
                while time.time() - start_time < 300:
                    poll = requests.get(
                        f"{CONFIG['UPSTREAM']}/api/result/get",
                        params={"job_id": job_id},
                        headers=spoofed_headers, # 保持一致的Header
                        proxies=proxies,
                        timeout=10
                    )
                    p_data = poll.json()
                    
                    if p_data.get("code") == 200 and p_data.get("result", {}).get("image_url"):
                        url = p_data["result"]["image_url"][0]
                        
                        # 更新统计与历史
                        full_data = load_data()
                        full_data["stats"]["total_calls"] += 1
                        full_data["stats"]["success_calls"] += 1
                        full_data["stats"]["last_call_time"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        full_data["history"].insert(0, {"prompt": prompt, "url": url, "time": datetime.now().strftime("%H:%M:%S")})
                        full_data["history"] = full_data["history"][:50]
                        save_data(full_data)

                        # 返回OpenAI兼容格式
                        chunk = {
                            "id": f"chatcmpl-{uuid.uuid4()}",
                            "object": "chat.completion.chunk",
                            "created": int(time.time()),
                            "model": body.get("model", "nano_banana"),
                            "choices": [{"index": 0, "delta": {"content": f"\n\n![Result]({url})"}, "finish_reason": "stop"}]
                        }
                        yield f"data: {json.dumps(chunk)}\n\n"
                        break
                    elif p_data.get("code") == 202:
                        pass
                    time.sleep(3)
                else:
                    # 超时
                    yield debug_chunk(">>> [错误] 渲染超时，请重试")
            else:
                # 尝试提取具体错误信息
                err_msg = "上游服务器返回错误"
                if res_json.get("message"):
                    if isinstance(res_json["message"], dict):
                        err_msg = res_json["message"].get("zh", res_json["message"].get("en", str(res_json["message"])))
                    else:
                        err_msg = str(res_json["message"])
                
                # 记录失败统计
                full_data = load_data()
                full_data["stats"]["total_calls"] += 1
                full_data["stats"]["failed_calls"] += 1
                save_data(full_data)
                
                yield debug_chunk(f">>> [拒绝] {err_msg} (Code: {res_json.get('code')})")

        except Exception as e:
            yield debug_chunk(f">>> [异常] {str(e)}")
        
        yield "data: [DONE]\n\n"

    return Response(stream_with_context(generate()), mimetype='text/event-stream')

# =================================================================
# 终极原生感 UI (HTML/CSS/JS)
# =================================================================
@app.route('/')
def index():
    html_template = """
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>{{PROJECT_NAME}}</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;900&family=Noto+Sans+SC:wght@300;400;700&display=swap');
        
        /* 主题变量定义 */
        :root {
            --bg: #0F0F12;
            --sidebar: #16161D;
            --card: #1C1C26;
            --border: rgba(255, 255, 255, 0.08);
            --primary: #FFBF00;
            --primary-glow: rgba(255, 191, 0, 0.3);
            --text: #FFFFFF;
            --text-dim: #8E8E93;
            --titlebar: #0F0F12;
        }

        [data-theme="deepsea"] {
            --bg: #050B14; --sidebar: #0A1628; --card: #0F2038; --primary: #007AFF; --primary-glow: rgba(0, 122, 255, 0.3);
        }

        [data-theme="cyber"] {
            --bg: #0D0216; --sidebar: #1A042D; --card: #260642; --primary: #BF00FF; --primary-glow: rgba(191, 0, 255, 0.3);
        }

        body {
            margin: 0; padding: 0; background: var(--bg); color: var(--text);
            font-family: 'Inter', 'Noto Sans SC', sans-serif; height: 100vh;
            display: flex; flex-direction: column; overflow-y: auto; /* 启用垂直滚动 */
            user-select: none;
            border: 1px solid var(--border); /* 窗口边框，便于拖拽调整大小 */
            box-sizing: border-box;
        }

        /* 原生感标题栏 */
        .title-bar {
            height: 38px; background: var(--titlebar); display: flex;
            justify-content: space-between; align-items: center;
            padding: 0 15px; -webkit-app-region: drag; /* 允许拖拽窗口 */
            border-bottom: 1px solid var(--border); z-index: 9999;
        }

        .title-bar .app-info { display: flex; align-items: center; gap: 10px; font-size: 12px; font-weight: 600; color: var(--text-dim); }
        .title-bar .controls { display: flex; gap: 5px; -webkit-app-region: no-drag; }
        .control-btn {
            width: 32px; height: 24px; display: flex; align-items: center; justify-content: center;
            border-radius: 4px; cursor: pointer; transition: 0.2s;
        }
        .control-btn:hover { background: rgba(255,255,255,0.1); }
        .control-btn.close:hover { background: #FF3B30; }

        /* 布局架构 */
        .app-container { flex: 1; display: flex; overflow: auto; }

        .sidebar {
            width: 360px; background: var(--sidebar); border-right: 1px solid var(--border);
            display: flex; flex-direction: column; padding: 20px; box-sizing: border-box;
            overflow-y: auto; /* 侧边栏垂直滚动 */
        }

        .main-view { flex: 1; display: flex; flex-direction: column; background: var(--bg); padding: 20px; position: relative; overflow-y: auto; min-height: 0; }

        /* 高级卡片 */
        .card {
            background: var(--card); border: 1px solid var(--border);
            border-radius: 14px; padding: 16px; margin-bottom: 16px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.2);
        }

        .label { font-size: 10px; font-weight: 800; color: var(--text-dim); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 8px; display: block; }

        /* 可复制的输入框 */
        .copy-box {
            background: #000; border: 1px solid #333; border-radius: 8px;
            padding: 10px; display: flex; justify-content: space-between; align-items: center;
            font-family: 'Fira Code', monospace; font-size: 11px; color: var(--primary); margin-bottom: 8px;
        }
        .copy-btn { cursor: pointer; opacity: 0.6; transition: 0.2s; padding: 4px; }
        .copy-btn:hover { opacity: 1; color: #fff; }

        /* 统计网格 */
        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .stat-card { background: rgba(0,0,0,0.2); padding: 12px; border-radius: 10px; border: 1px solid var(--border); }
        .stat-val { font-size: 20px; font-weight: 900; color: var(--primary); }
        .stat-lbl { font-size: 10px; color: var(--text-dim); margin-top: 4px; }

        /* 交互组件 */
        select, textarea {
            width: 100%; background: rgba(0,0,0,0.3); border: 1px solid #333; color: #fff;
            padding: 12px; border-radius: 10px; font-family: inherit; margin-bottom: 12px; outline: none; transition: 0.3s;
        }
        select:focus, textarea:focus { border-color: var(--primary); box-shadow: 0 0 0 3px var(--primary-glow); }

        .upload-zone {
            border: 2px dashed #444; border-radius: 12px; padding: 25px 10px; text-align: center;
            cursor: pointer; transition: 0.3s; background: rgba(255,255,255,0.02); margin-bottom: 12px;
        }
        .upload-zone:hover { border-color: var(--primary); background: rgba(255, 191, 0, 0.05); }
        #preview-img { max-width: 100%; max-height: 150px; border-radius: 8px; display: none; margin: 0 auto; }

        .btn-action {
            width: 100%; padding: 14px; background: var(--primary); color: #000; border: none;
            border-radius: 12px; font-weight: 900; font-size: 14px; cursor: pointer; transition: 0.3s;
        }
        .btn-action:hover { transform: translateY(-2px); box-shadow: 0 8px 20px var(--primary-glow); }

        /* 终端与画廊 */
        .terminal-container { flex: 1; display: flex; flex-direction: column; background: #050505; border: 1px solid var(--border); border-radius: 16px; overflow: hidden; }
        .terminal-header { background: #111; padding: 10px 20px; display: flex; justify-content: space-between; font-size: 11px; color: var(--text-dim); }
        .terminal-body { flex: 1; padding: 15px; overflow-y: auto; font-family: 'Fira Code', monospace; font-size: 12px; line-height: 1.6; }
        
        .gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(70px, 1fr)); gap: 8px; }
        .gallery-item { aspect-ratio: 1; border-radius: 8px; overflow: hidden; border: 1px solid #333; cursor: pointer; transition: 0.2s; }
        .gallery-item:hover { border-color: var(--primary); transform: scale(1.05); }
        .gallery-item img { width: 100%; height: 100%; object-fit: cover; }

        .theme-selector { display: flex; gap: 8px; margin-top: 10px; }
        .theme-dot { width: 16px; height: 16px; border-radius: 50%; cursor: pointer; border: 2px solid transparent; }
        .theme-dot.active { border-color: #fff; }

        /* Cherry Studio 风格日志卡片 */
        .log-card {
            background-color: #FFF0F0;
            border-radius: 8px;
            padding: 12px 16px;
            margin-bottom: 8px;
            border: 1px solid #E0C0C0;
            font-family: 'Consolas', monospace;
            font-size: 14px;
            line-height: 1.6;
            position: relative;
        }
        .log-card.debug {
            background-color: #F0F8FF;
            border-color: #C0D0E0;
        }
        .log-card.error {
            background-color: #FFF0F0;
            border-color: #E0C0C0;
        }
        .log-card .log-content {
            margin-right: 80px;
            word-break: break-all;
            white-space: pre-wrap;
        }
        .log-card .log-actions {
            position: absolute;
            right: 12px;
            top: 12px;
            display: flex;
            gap: 8px;
        }
        .log-card .log-detail {
            color: #0066CC;
            cursor: pointer;
            font-size: 12px;
            text-decoration: none;
        }
        .log-card .log-close {
            color: #999;
            cursor: pointer;
            font-size: 16px;
            line-height: 1;
        }
        .log-card .log-close:hover {
            color: #D00;
        }

        /* 成功提示 */
        #toast {
            position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%);
            background: var(--primary); color: #000; padding: 8px 20px; border-radius: 20px;
            font-size: 12px; font-weight: 700; display: none; z-index: 10000;
        }
    </style>
</head>
<body data-theme="obsidian">
    <!-- 原生标题栏 -->
    <div class="title-bar">
        <div class="app-info">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="var(--primary)"><path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"></path></svg>
            {{PROJECT_NAME}} v{{VERSION}}
        </div>
        <div class="controls">
            <div class="control-btn" onclick="window.pywebview.api.minimize()">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="5" y1="12" x2="19" y2="12"></line></svg>
            </div>
            <div class="control-btn" onclick="window.pywebview.api.toggle_maximize()">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" stroke="currentColor" stroke-width="2" fill="none"/></svg>
            </div>
            <div class="control-btn close" onclick="window.pywebview.api.close()">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </div>
        </div>
    </div>

    <div class="app-container">
        <!-- 侧边栏 -->
        <div class="sidebar">
            <div class="card">
                <span class="label">数据看板</span>
                <div class="stats-grid">
                    <div class="stat-card">
                        <div class="stat-val" id="stat-total">0</div>
                        <div class="stat-lbl">累计请求</div>
                    </div>
                    <div class="stat-card">
                        <div class="stat-val" id="stat-success">0</div>
                        <div class="stat-lbl">成功渲染</div>
                    </div>
                </div>
                <div style="margin-top:12px; font-size:10px; color:var(--text-dim);">最近活动: <span id="stat-last" style="color:#fff;">-</span></div>
            </div>

            <div class="card">
                <span class="label">本地节点 (点击复制)</span>
                <div class="copy-box">
                    <span id="node-url">http://127.0.0.1:{{PORT}}</span>
                    <div class="copy-btn" onclick="copyText('node-url')">
                        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg>
                    </div>
                </div>
                <span class="label">API KEY</span>
                <div class="copy-box">
                    <span id="api-key">{{API_KEY}}</span>
                    <div class="copy-btn" onclick="copyText('api-key')">
                        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg>
                    </div>
                </div>
            </div>

            <div class="card" style="flex:1; overflow-y:auto;">
                <span class="label">历史画廊</span>
                <div class="gallery" id="historyGallery"></div>
            </div>

            <div class="card">
                <span class="label">外观主题</span>
                <div class="theme-selector">
                    <div class="theme-dot active" style="background:#FFBF00;" onclick="changeTheme('obsidian', this)"></div>
                    <div class="theme-dot" style="background:#007AFF;" onclick="changeTheme('deepsea', this)"></div>
                    <div class="theme-dot" style="background:#BF00FF;" onclick="changeTheme('cyber', this)"></div>
                </div>
            </div>
        </div>

        <!-- 主视图 -->
        <div class="main-view">
            <div class="card" style="margin-bottom:20px;">
                <span class="label">任务配置</span>
                <div style="display:flex; gap:10px;">
                    <select id="modelSelect" style="flex:1;">
                        <option value="nano_banana">Nano Banana (极速)</option>
                        <option value="standard">Standard (高清)</option>
                    </select>
                    <select id="ratioSelect" style="flex:1;">
                        <option value="match_input_image">原始比例</option>
                        <option value="1:1">1:1 正方</option>
                        <option value="3:2">3:2 横向</option>
                        <option value="2:3">2:3 纵向</option>
                        <option value="9:16">9:16 竖屏</option>
                        <option value="16:9">16:9 宽屏</option>
                        <option value="3:4">3:4 纵向</option>
                        <option value="4:3">4:3 横向</option>
                    </select>
                </div>
                
                <div class="upload-zone" id="dropZone">
                    <div id="uploadPrompt">拖拽、点击或粘贴参考图</div>
                    <img id="preview-img">
                    <input type="file" id="fileInput" hidden accept="image/*">
                </div>

                <textarea id="promptInput" rows="2" placeholder="输入提示词..."></textarea>
                <button class="btn-action" id="genBtn">执行渲染任务</button>
            </div>

            <div class="terminal-container">
                <div class="terminal-header">
                    <span>CORE TERMINAL</span>
                    <span id="statusText">READY</span>
                </div>
                <div class="terminal-body" id="terminalOut">
                    <div style="color:#444;">> 系统内核已就绪，等待指令...</div>
                </div>
            </div>
        </div>
    </div>

    <div id="toast">已复制到剪贴板</div>

    <script>
        let currentBase64 = null;

        // 复制功能
        function copyText(id) {
            const text = document.getElementById(id).innerText;
            const el = document.createElement('textarea');
            el.value = text;
            document.body.appendChild(el);
            el.select();
            document.execCommand('copy');
            document.body.removeChild(el);
            
            const toast = document.getElementById('toast');
            toast.style.display = 'block';
            setTimeout(() => toast.style.display = 'none', 2000);
        }

        // 主题切换
        function changeTheme(theme, el) {
            document.body.setAttribute('data-theme', theme);
            document.querySelectorAll('.theme-dot').forEach(d => d.classList.remove('active'));
            el.classList.add('active');
            fetch('/api/theme', { method: 'POST', headers: {'Content-Type': 'application/json'}, body: JSON.stringify({theme}) });
        }

        // 上传逻辑
        const dropZone = document.getElementById('dropZone');
        const fileInput = document.getElementById('fileInput');
        const previewImg = document.getElementById('preview-img');
        const uploadPrompt = document.getElementById('uploadPrompt');

        dropZone.onclick = () => fileInput.click();
        const processFile = (file) => {
            if (!file || !file.type.startsWith('image/')) return;
            const reader = new FileReader();
            reader.onload = (e) => {
                currentBase64 = e.target.result;
                previewImg.src = currentBase64;
                previewImg.style.display = 'block';
                uploadPrompt.style.display = 'none';
            };
            reader.readAsDataURL(file);
        };
        fileInput.onchange = (e) => processFile(e.target.files[0]);
        window.addEventListener('paste', (e) => {
            const items = e.clipboardData.items;
            for (let item of items) {
                if (item.type.indexOf('image') !== -1) processFile(item.getAsFile());
            }
        });

        // 数据刷新
        const refreshData = async () => {
            const res = await fetch('/api/data');
            const data = await res.json();
            document.getElementById('stat-total').innerText = data.stats.total_calls;
            document.getElementById('stat-success').innerText = data.stats.success_calls;
            document.getElementById('stat-last').innerText = data.stats.last_call_time;
            
            const gallery = document.getElementById('historyGallery');
            gallery.innerHTML = data.history.map(item => `
                <div class="gallery-item" onclick="window.open('${item.url}')">
                    <img src="${item.url}" title="${item.prompt}">
                </div>
            `).join('');

            if(data.settings.theme) {
                document.body.setAttribute('data-theme', data.settings.theme);
                document.querySelectorAll('.theme-dot').forEach(d => {
                    if(d.getAttribute('onclick').includes(data.settings.theme)) d.classList.add('active');
                    else d.classList.remove('active');
                });
            }
        };
        refreshData();

        // 加载可用模型
        const loadModels = async () => {
            try {
                const res = await fetch('/v1/models');
                const data = await res.json();
                const modelSelect = document.getElementById('modelSelect');
                
                // 清空现有选项（保留第一个作为默认）
                modelSelect.innerHTML = '';
                
                // 添加模型选项
                data.data.forEach(model => {
                    const option = document.createElement('option');
                    option.value = model.id;
                    
                    // 使用API返回的显示名称，如果没有则使用模型ID
                    option.text = model.display_name || model.id;
                    
                    // 标记支持图像的模型
                    if (model.supports_images) {
                        option.text += ' 📷';
                    }
                    
                    modelSelect.appendChild(option);
                });
                
                // 如果没有模型，添加默认选项
                if (data.data.length === 0) {
                    const option = document.createElement('option');
                    option.value = 'nano_banana';
                    option.text = 'Nano Banana (极速)';
                    modelSelect.appendChild(option);
                }
                
                console.log('已加载模型列表:', data.data.length, '个模型');
            } catch (error) {
                console.error('加载模型失败:', error);
                // 保留默认选项
            }
        };
        
        // 页面加载时获取模型列表
        loadModels();

        // 日志输出 - Cherry Studio风格
        const addLog = (tag, msg) => {
            const out = document.getElementById('terminalOut');
            const card = document.createElement('div');
            card.className = `log-card ${tag.toLowerCase()}`;
            
            const content = document.createElement('div');
            content.className = 'log-content';
            content.textContent = '[' + new Date().toLocaleTimeString() + '] ' + msg;
            
            const actions = document.createElement('div');
            actions.className = 'log-actions';
            
            const detailLink = document.createElement('span');
            detailLink.className = 'log-detail';
            detailLink.textContent = '详情';
            detailLink.onclick = () => {
                // 可以展开/折叠详细信息，这里暂时只是复制消息
                navigator.clipboard.writeText(msg).then(() => {
                    const toast = document.getElementById('toast');
                    toast.textContent = '已复制消息到剪贴板';
                    toast.style.display = 'block';
                    setTimeout(() => toast.style.display = 'none', 2000);
                });
            };
            
            const closeBtn = document.createElement('span');
            closeBtn.className = 'log-close';
            closeBtn.textContent = '×';
            closeBtn.onclick = () => card.remove();
            
            actions.appendChild(detailLink);
            actions.appendChild(closeBtn);
            card.appendChild(content);
            card.appendChild(actions);
            out.appendChild(card);
            out.scrollTop = out.scrollHeight;
        };

        // 任务提交
        document.getElementById('genBtn').onclick = async () => {
            const prompt = document.getElementById('promptInput').value;
            if (!prompt) return alert("请输入提示词！");

            const btn = document.getElementById('genBtn');
            const status = document.getElementById('statusText');
            const out = document.getElementById('terminalOut');

            btn.disabled = true;
            out.innerHTML = '';
            status.innerText = 'BUSY';

            try {
                const res = await fetch('/v1/chat/completions', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        model: document.getElementById('modelSelect').value,
                        aspect_ratio: document.getElementById('ratioSelect').value,
                        messages: [{
                            role: 'user',
                            content: [
                                { type: 'text', text: prompt },
                                { type: 'image_url', image_url: { url: currentBase64 } }
                            ]
                        }]
                    })
                });

                const reader = res.body.getReader();
                const decoder = new TextDecoder();

                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    const chunk = decoder.decode(value);
                    const lines = chunk.split('\\n');
                    
                    for (let line of lines) {
                        if (line.startsWith('data: ')) {
                            try {
                                const data = JSON.parse(line.substring(6));
                                // 处理调试消息和结果
                                if (data.choices && data.choices[0].delta.content) {
                                    const content = data.choices[0].delta.content;
                                    // 检查是否是调试消息（以 >>> 开头）
                                    if (content.startsWith('>>>')) {
                                        addLog('DEBUG', content);
                                    } else {
                                        // 尝试提取图片URL
                                        const urlMatch = content.match(/\\((.*?)\\)/);
                                        if (urlMatch) {
                                            const url = urlMatch[1];
                                            out.innerHTML += '<div style="margin-top:15px;text-align:center;"><img src="' + url + '" style="max-width:100%;border-radius:10px;border:1px solid var(--primary);"></div>';
                                            status.innerText = 'DONE';
                                            refreshData();
                                        } else {
                                            // 其他文本内容
                                            addLog('INFO', content);
                                        }
                                    }
                                }
                                if (data.error) {
                                    addLog('ERROR', data.error.message);
                                    throw new Error(data.error.message);
                                }
                            } catch(e) {}
                        }
                    }
                }
            } catch (e) {
                addLog('ERROR', e.message);
                status.innerText = 'FAIL';
            } finally {
                btn.disabled = false;
            }
        };
    </script>
</body>
</html>
"""
    content = html_template.replace("{{PROJECT_NAME}}", CONFIG["PROJECT_NAME"])
    content = content.replace("{{PORT}}", str(CONFIG["PORT"]))
    content = content.replace("{{API_KEY}}", CONFIG["API_KEY"])
    content = content.replace("{{VERSION}}", CONFIG["VERSION"])
    return content

# =================================================================
# 启动入口 (Desktop App Entry)
# =================================================================
class Api:
    def __init__(self):
        self.maximized = False

    def close(self):
        window.destroy()
    def minimize(self):
        window.minimize()
    def maximize(self):
        window.maximize()
        self.maximized = True
    def restore(self):
        window.restore()
        self.maximized = False
    def toggle_maximize(self):
        if self.maximized:
            self.restore()
        else:
            self.maximize()

def run_flask():
    app.run(port=CONFIG["PORT"], threaded=True)

if __name__ == "__main__":
    # 1. 启动 Flask
    t = threading.Thread(target=run_flask)
    t.daemon = True
    t.start()

    # 2. 创建原生窗口 (无边框模式)
    api = Api()
    window = webview.create_window(
        CONFIG["PROJECT_NAME"], 
        f"http://127.0.0.1:{CONFIG['PORT']}",
        width=1280,
        height=820,
        frameless=True,  # 开启无边框模式，实现原生高级感
        easy_drag=True,
        resizable=True,  # 允许调整窗口大小
        min_size=(800, 600),  # 最小窗口尺寸
        background_color='#0F0F12',
        js_api=api
    )
    
    # 3. 启动
    webview.start()


然后bat脚本如下你看看：
@echo off
setlocal enabledelayedexpansion
chcp 65001 >nul 2>&1
title Project Chimera: Zaiwen 2API - 智能启动器

:: ==========================================
:: Project Chimera - Smart Launcher (Chinese Edition)
:: 功能:
::   - 自动检测/下载 Python
::   - 自动创建虚拟环境
::   - 实时依赖库安装日志
::   - 极速启动模式 (Marker File)
:: ==========================================

cd /d "%~dp0"

:: 配置
set "APP_NAME=Zaiwen 2API 智能服务"
set "PYTHON_VERSION=3.11.9"
set "PYTHON_DIR=%~dp0python"
set "VENV_DIR=%~dp0venv"
set "MARKER_FILE=%~dp0.env_ready"
set "PYTHON_URL=https://www.python.org/ftp/python/3.11.9/python-3.11.9-embed-amd64.zip"
set "GET_PIP_URL=https://bootstrap.pypa.io/get-pip.py"

:: 显示标题
echo.
echo ==========================================
echo    %APP_NAME% - 智能启动器
echo ==========================================
echo.

:: 极速检查 - 如果标记文件存在，跳过完整检查
if exist "%MARKER_FILE%" (
    echo [*] 极速模式: 环境已在之前验证通过
    echo.
    goto :run_app
)

echo [*] 初次运行或环境需要检查...
echo.

:: ==========================================
:: 步骤 1: 检查 Python 环境
:: ==========================================
echo [1/4] 正在检查 Python 环境...

set "PYTHON_EXE="
set "USE_EMBEDDED=0"

:: 优先级 1: 检查嵌入式 Python
if exist "%PYTHON_DIR%\python.exe" (
    set "PYTHON_EXE=%PYTHON_DIR%\python.exe"
    set "USE_EMBEDDED=1"
    echo      [+] 发现嵌入式 Python
    goto :python_found
)

:: 优先级 2: 检查系统 Python
where python >nul 2>&1
if %errorlevel% equ 0 (
    for /f "tokens=2 delims= " %%v in ('python --version 2^>^&1') do set "SYSTEM_PY_VER=%%v"
    echo      [+] 发现系统 Python: !SYSTEM_PY_VER!
    
    :: 检查版本 >= 3.8
    for /f "tokens=1,2 delims=." %%a in ("!SYSTEM_PY_VER!") do (
        if %%a geq 3 if %%b geq 8 (
            set "PYTHON_EXE=python"
            echo      [+] 版本符合要求，使用系统 Python
            goto :python_found
        )
    )
    echo      [-] 版本过低，需要 Python 3.8+
)

:: 没有找到合适的 Python，下载嵌入式版本
echo      [-] 未找到合适的 Python，正在下载嵌入式版本...
goto :download_python

:python_found
echo      [OK] Python 环境就绪
echo.
goto :check_venv

:: ==========================================
:: 步骤 2: 下载嵌入式 Python
:: ==========================================
:download_python
echo.
echo [*] 正在下载 Python %PYTHON_VERSION% 嵌入版...
echo     URL: %PYTHON_URL%
echo.

:: 创建 python 目录
if not exist "%PYTHON_DIR%" mkdir "%PYTHON_DIR%"

:: 使用 PowerShell 下载 (带进度条)
set "PYTHON_ZIP=%PYTHON_DIR%\python.zip"
echo     正在下载...
powershell -Command "& {$ProgressPreference = 'Continue'; [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; Invoke-WebRequest -Uri '%PYTHON_URL%' -OutFile '%PYTHON_ZIP%' -UseBasicParsing}"

if not exist "%PYTHON_ZIP%" (
    echo.
    echo [错误] Python 下载失败。请检查网络连接。
    echo         您可以手动安装 Python 3.8+ 并重试。
    pause
    exit /b 1
)

:: 解压
echo     正在解压...
powershell -Command "& {Expand-Archive -Path '%PYTHON_ZIP%' -DestinationPath '%PYTHON_DIR%' -Force}"
del "%PYTHON_ZIP%" 2>nul

:: 启用 pip 支持 - 修改 python311._pth
set "PTH_FILE=%PYTHON_DIR%\python311._pth"
if exist "%PTH_FILE%" (
    echo python311.zip> "%PTH_FILE%"
    echo .>> "%PTH_FILE%"
    echo Lib\site-packages>> "%PTH_FILE%"
    echo import site>> "%PTH_FILE%"
)

:: 下载并安装 pip
echo.
echo [*] 正在安装 pip...
set "GET_PIP=%PYTHON_DIR%\get-pip.py"
powershell -Command "& {$ProgressPreference = 'Continue'; [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; Invoke-WebRequest -Uri '%GET_PIP_URL%' -OutFile '%GET_PIP%' -UseBasicParsing}"

"%PYTHON_DIR%\python.exe" "%GET_PIP%"
del "%GET_PIP%" 2>nul

set "PYTHON_EXE=%PYTHON_DIR%\python.exe"
set "USE_EMBEDDED=1"
echo      [OK] 嵌入式 Python 安装完成
echo.

:: ==========================================
:: 步骤 3: 检查/创建虚拟环境
:: ==========================================
:check_venv
echo [2/4] 正在检查虚拟环境...

:: 对于嵌入式 Python，跳过 venv (直接使用嵌入式环境)
if "%USE_EMBEDDED%"=="1" (
    echo      [+] 使用嵌入式 Python，跳过虚拟环境创建
    set "PIP_EXE=%PYTHON_DIR%\Scripts\pip.exe"
    if not exist "!PIP_EXE!" set "PIP_EXE=%PYTHON_DIR%\python.exe -m pip"
    echo.
    goto :check_deps
)

:: 检查 venv 是否存在
if exist "%VENV_DIR%\Scripts\python.exe" (
    echo      [OK] 虚拟环境已存在
    set "PYTHON_EXE=%VENV_DIR%\Scripts\python.exe"
    set "PIP_EXE=%VENV_DIR%\Scripts\pip.exe"
    echo.
    goto :check_deps
)

:: 创建虚拟环境
echo      [+] 正在创建虚拟环境...
python -m venv "%VENV_DIR%"
if %errorlevel% neq 0 (
    echo [错误] 虚拟环境创建失败
    pause
    exit /b 1
)

set "PYTHON_EXE=%VENV_DIR%\Scripts\python.exe"
set "PIP_EXE=%VENV_DIR%\Scripts\pip.exe"
echo      [OK] 虚拟环境创建成功
echo.

:: ==========================================
:: 步骤 4: 检查/安装依赖库
:: ==========================================
:check_deps
echo [3/4] 正在检查依赖库...

:: 快速检查 - 尝试导入关键模块
"%PYTHON_EXE%" -c "import fastapi; import uvicorn; import httpx; import loguru; import aiosqlite; import PySide6; import multipart" 2>nul
if %errorlevel% equ 0 (
    echo      [OK] 所有依赖库已安装
    echo.
    goto :create_marker
)

:: 安装缺失的依赖库 (显示完整输出以便查看)
echo.
echo      [-] 发现缺失依赖，正在安装...
echo      ============================================
echo.

if "%USE_EMBEDDED%"=="1" (
    echo [pip] 正在使用 requirements.txt 安装...
    "%PYTHON_EXE%" -m pip install -r requirements.txt --no-warn-script-location
) else (
    echo [pip] 正在使用 requirements.txt 安装...
    "%PIP_EXE%" install -r requirements.txt
)

echo.
if %errorlevel% neq 0 (
    echo [错误] 部分依赖安装失败
    echo         请检查网络连接或更换 PyPI 源。
    pause
    exit /b 1
)
echo      ============================================
echo      [OK] 依赖库安装成功！
echo.

:: ==========================================
:: 步骤 5: 创建标记文件
:: ==========================================
:create_marker
echo [4/4] 正在完成设置...

:: 创建带时间戳的标记文件
echo Environment validated on %date% %time%> "%MARKER_FILE%"
echo Python: %PYTHON_EXE%>> "%MARKER_FILE%"
echo      [OK] 环境准备就绪
echo.

:: ==========================================
:: 运行应用程序
:: ==========================================
:run_app
echo ==========================================
echo    正在启动 Zaiwen 2API 服务...
echo    服务地址: http://127.0.0.1:8000
echo    请勿关闭此窗口
echo ==========================================
echo.

:: 确定 Python 执行路径 (再次确认以防万一)
if exist "%VENV_DIR%\Scripts\python.exe" (
    set "PYTHON_EXE=%VENV_DIR%\Scripts\python.exe"
) else if exist "%PYTHON_DIR%\python.exe" (
    set "PYTHON_EXE=%PYTHON_DIR%\python.exe"
) else (
    set "PYTHON_EXE=python"
)

:: 运行主程序
"%PYTHON_EXE%" main.py

if %errorlevel% neq 0 (
    echo.
    echo [错误] 应用程序异常退出，错误代码: %errorlevel%
    echo.
    :: 删除标记文件以强制下次重新检查
    del "%MARKER_FILE%" 2>nul
    pause
)

endlocal

这样一切都是为了方便，请开始你的任务，请你用中文回复我












```










