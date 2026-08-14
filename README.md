# Web to Figma (Chrome Extension)

## Demo

[Watch Demo](https://x.com/xin_pai88825/status/2029792800653594675)

## Extension Download & Tutorial

[Download & User Guide](https://gwrdluzl9j9.feishu.cn/wiki/TTibw3e8niWpvmkfSIWc8bUmnJc)

Capture web pages and export them as data for Figma workflows.  
Supports full-page capture, element capture, and optional cross-origin image proxy fetching.

## Features

- In-page floating toolbar with one-click capture
- Optional cross-origin image proxy mode to reduce missing images
- Configurable image fetch concurrency (`4/6/8/10/12/16/20/infinite`)
- Export capture result as `.json`

## Project Structure

```text
.
├── manifest.json
├── background.js
├── capture.js
├── runner.js
├── inpage-toolbar.js
├── popup.html
├── popup.css
├── popup.js
└── logo/
```

## Local Installation (Developer Mode)

1. Open `chrome://extensions/`
2. Enable `Developer mode`
3. Click `Load unpacked`
4. Select this repository root directory

## Usage

1. Open any webpage
2. Click the extension icon (or use the in-page toolbar)
3. Enable `Cross-origin image proxy mode` if needed
4. Click `Start Capture`
5. Download the generated `figma-capture-*.json`

## 使用方式二：脚本

(async () => {
  const sleep = (ms) => new Promise(r => setTimeout(r, ms));

  // 1) 注入 capture.js
  if (!window.figma?.captureForDesign) {
    const r = await fetch("https://mcp.figma.com/mcp/html-to-design/capture.js");
    const s = await r.text();
    const el = document.createElement("script");
    el.textContent = s;
    document.head.appendChild(el);
    await sleep(1200);
  }

  // 2) 触发懒加载：滚动到底再回顶
  const step = Math.max(400, Math.floor(window.innerHeight * 0.8));
  for (let y = 0; y < document.body.scrollHeight; y += step) {
    window.scrollTo(0, y);
    await sleep(180);
  }
  await sleep(600);
  window.scrollTo(0, 0);

  // 3) 等图片与字体
  const imgs = Array.from(document.images || []);
  await Promise.allSettled(
    imgs.map(img => img.complete ? Promise.resolve() : new Promise(res => {
      img.addEventListener("load", res, { once: true });
      img.addEventListener("error", res, { once: true });
      setTimeout(res, 4000);
    }))
  );
  if (document.fonts?.ready) await Promise.race([document.fonts.ready, sleep(3000)]);
  await sleep(500);

  // 4) 复制模式抓取
  return await window.figma.captureForDesign({
    selector: "body"
  });
})();

复制脚本后，打开任意网页，单击右键，选择检查
[图片]
切换到控制，在底部粘贴脚本，然后按回车即可。
注：首次执行的时候，浏览器会让你手动输入“”允许粘贴“”的文本，输入后按回车，再粘贴脚本即可
[图片]

## Cross-Origin Image Proxy Notes

- When enabled, the extension fetches images through the background proxy.
- This improves image completeness but can make capture slower.
- Use `Image Fetch Concurrency` to balance speed and stability.

## Packaging

Run in repository root:

```bash
zip -r web-to-figma-extension.zip . -x "*.DS_Store" -x ".git/*"
```

## Notes

- `capture.js` is the core capture runtime.
- If you obfuscate code, do it on a release copy, not on source files.

## Disclaimer

- This project is provided for learning, research, and productivity use only.
- You are responsible for complying with website terms, copyright rules, privacy laws, and applicable local regulations.
- Do not use this tool to capture or distribute unauthorized, sensitive, or illegal content.
- The authors and contributors are not liable for misuse, data loss, or any direct/indirect damages caused by this project.

## License

This project is open-sourced under the [MIT License](./LICENSE).
