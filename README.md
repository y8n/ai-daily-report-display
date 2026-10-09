# ai-daily-report-display

**每日 AI 资讯**的语音播报页：一边放语音，一边按真实时间轴滚动高亮字幕。

## 用法

    https://y8n.github.io/ai-daily-report-display/?date=YYYY-MM-DD

一个页面服务所有日期 —— 换日期只改 `?date=`，**不需要每天新建页面**。
页面上也有「前一天 / 后一天」和日期选择器。

## 它怎么工作

```
index.html              通用播放器（唯一一份）
data/<日期>.json         当天字幕：audio 地址 + duration + 逐句 [t0,t1,text]
        ↓
   <audio> 指向 R2 上的 mp3
```

- **字幕数据放本站**（`data/`）。GitHub Pages 与 R2 **不同源**，
  而 R2 默认不返回 `Access-Control-Allow-Origin`，跨域 `fetch` 会被浏览器拦掉——
  症状是「页面能开、音频能放，但字幕永远不出来」。放同源就绕开了。
- **音频留在 R2**：`<audio>` 这类媒体元素不吃 CORS，所以语音不必进仓库（仓库因此不会变大）。
- 页面会**先试同源 `./data/`，失败再试 R2**，所以同一份页面在 R2 上也照样能用。

时间轴是 **edge-tts 的 WordBoundary 真实时间戳**聚合成的句级时间点，不是按字数估算的。

## 这个仓库的内容是生成的

`index.html` 与 `data/*.json` 由本地 skill 产出：

- 页面源文件：`~/.dsh/skills/daily-ai-digest/assets/voice-player.html`
- 每日流程：`publish_audio.py` 合成音频 → 上传 R2 → 生成并写入本仓库 `data/<日期>.json` → 提交推送

**不要手改 `index.html`**（会被下次同步覆盖）。要改播放器，改 skill 里那份再同步过来。

## 本地预览

    cd ~/workspace/ai-daily-report-display
    python3 -m http.server 8080
    # 打开 http://localhost:8080/?date=2026-10-09
