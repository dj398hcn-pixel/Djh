<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>MindSpace Wellness</title>
  
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    // 【调整 1】将 import 替换为从全局 React 对象中获取
    const { useState, useEffect, useRef } = React;

    // ── Fonts via Google Fonts injected in head ──────────────────────────────────
    const fontLink = document.createElement("link");
    fontLink.rel = "stylesheet";
    fontLink.href =
      "https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;1,400&family=DM+Sans:wght@300;400;500&display=swap";
    document.head.appendChild(fontLink);

    // ── Global styles ────────────────────────────────────────────────────────────
    const globalStyle = document.createElement("style");
    globalStyle.textContent = `
      *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
      :root {
        --sage:      #7aab8a;
        --sage-light:#b6d8c2;
        --sage-pale: #e8f4ec;
        --warm:      #f5f0e8;
        --cream:     #fdfaf5;
        --clay:      #c8856a;
        --clay-light:#f0d5ca;
        --slate:     #4a5568;
        --dusk:      #2d3748;
        --mist:      #edf2f7;
        --gold:      #d4a94a;
        --lavender:  #9b8ec4;
        --sky:       #6baed6;
        --r: .75rem;
        --R: 1.25rem;
      }
      body { background: var(--cream); font-family: 'DM Sans', sans-serif; color: var(--dusk); margin: 0; }
      h1,h2,h3,h4 { font-family: 'Lora', serif; }
      button { cursor: pointer; border: none; font-family: 'DM Sans', sans-serif; }
      textarea, input { font-family: 'DM Sans', sans-serif; }
      ::-webkit-scrollbar { width: 6px; }
      ::-webkit-scrollbar-track { background: var(--mist); }
      ::-webkit-scrollbar-thumb { background: var(--sage-light); border-radius:
