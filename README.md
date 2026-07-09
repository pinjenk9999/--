# -<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>圖片點擊自動編號工具</title>
<style>
body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    background: #eaeded;
    color: #333;
}
#bar {
    padding: 12px 20px;
    background: #1a1a1a;
    color: #fff;
    display: flex;
    gap: 15px;
    flex-wrap: wrap;
    align-items: center;
    box-shadow: 0 2px 10px rgba(0,0,0,0.3);
    position: relative;
    z-index: 100;
}
.brand {
    font-size: 18px;
    font-weight: bold;
    margin-right: 10px;
    letter-spacing: 1px;
}
.control-group {
    display: flex;
    align-items: center;
    gap: 8px;
    background: #2a2a2a;
    padding: 6px 12px;
    border-radius: 6px;
}
label {
    font-size: 14px;
    color: #ccc;
}
select, input[type="number"] {
    background: #3a3a3a;
    border: 1px solid #555;
    color: #fff;
    padding: 4px 8px;
    border-radius: 4px;
    outline: none;
    font-size: 14px;
}
input[type="number"] {
    width: 60px;
    text-align: center;
}
select {
    cursor: pointer;
}
/* 自定義檔案上傳按鈕 */
.file-upload {
    position: relative;
    display: inline-block;
    cursor: pointer;
    background: #0076d6;
    color: white;
    padding: 6px 14px;
    border-radius: 6px;
    font-size: 14px;
    font-weight: 500;
    transition: background 0.2s;
}
.file-upload:hover {
    background: #005ea1;
}
.file-upload input[type="file"] {
    position: absolute;
    left: 0;
    top: 0;
    opacity: 0;
    width: 100%;
    height: 100%;
    cursor: pointer;
}
button {
    background: #444;
    color: #fff;
    border: none;
    padding: 6px 14px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 14px;
    transition: background 0.2s;
}
button:hover {
    background: #555;
}
#clear {
    background: #d9383a;
}
#clear:hover {
    background: #b82e30;
}
#wrap {
    position: relative;
    display: inline-block;
    margin: 20px;
    border: 1px solid #ccc;
    background: #fff;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    border-radius: 4px;
    overflow: hidden;
}
#img {
    display: block;
    max-width: 100%;
    user-select: none;
    -webkit-user-drag: none;
}
/* 圓形紅底白字標籤，比照範例圖設計，完美防止重疊時看不清 */
.marker {
    position: absolute;
    transform: translate(-50%, -50%);
    width: 28px;
    height: 28px;
    line-height: 28px;
    background-color: #e53935;
    color: white;
    font-weight: bold;
    font-size: 14px;
    text-align: center;
    border-radius: 50%;
    box-shadow: 0 2px 6px rgba(0,0,0,0.4);
    border: 2px solid #fff;
    cursor: move;
    user-select: none;
    z-index: 50;
    transition: transform 0.1s;
}
.marker:hover {
    transform: translate(-50%, -50%) scale(1.1);
    background-color: #d32f2f;
}
#line {
    position: absolute;
    pointer-events: none;
    border-top: 2px dashed #00aaff;
    display: none;
    z-index: 40;
}
</style>
</head>
<body>
<div id="bar">
    <div class="brand">圖片自動編號工具</div>
    
    <div class="file-upload">
        <span>選擇圖片</span>
        <input type="file" id="upload" accept="image/*">
    </div>

    <div class="control-group">
        <label for="mode">模式</label>
        <select id="mode">
            <option value="click">單點點擊</option>
            <option value="line">畫線平均分散</option>
        </select>
    </div>

    <div class="control-group">
        <label for="start">起始編號</label>
        <input id="start" type="number" value="1" min="0">
    </div>

    <div class="control-group">
        <label for="count">劃線數量</label>
        <input id="count" type="number" value="10" min="1">
    </div>

    <button id="undo">復原 (Undo)</button>
    <button id="clear">全部清除</button>
</div>

<div id="wrap">
    <img id="img" alt="請先上傳圖片">
    <div id="line"></div>
</div>

<script>
const img = document.getElementById('img');
const wrap = document.getElementById('wrap');
const upload = document.getElementById('upload');
const mode = document.getElementById('mode');
const start = document.getElementById('start');
const count = document.getElementById('count');
const line = document.getElementById('line');
const undo = document.getElementById('undo');
const clear = document.getElementById('clear');

let n = 1;
let stack = [];
let drag = null;
let sx = 0, sy = 0;
let isDrawingLine = false;

// 上傳圖片處理
upload.onchange = e => {
    const f = e.target.files[0];
    if (!f) return;
    const r = new FileReader();
    r.onload = x => {
        img.src = x.target.result;
        n = parseInt(start.value, 10) || 1;
        clearAll();
    };
    r.readAsDataURL(f);
};

// 新增標籤函數
function add(x, y, num) {
    let d = document.createElement('div');
    d.className = 'marker';
    d.textContent = num;
    d.style.left = x + 'px';
    d.style.top = y + 'px';
    wrap.appendChild(d);
    stack.push(d);
    
    // 拖曳功能
    d.onmousedown = e => {
        drag = d;
        e.stopPropagation();
        e.preventDefault();
    };
}

// 點擊模式
wrap.onclick = e => {
    if (mode.value !== "click" || e.target !== img) return;
    const r = wrap.getBoundingClientRect();
    add(e.clientX - r.left, e.clientY - r.top, n++);
};

// 滑鼠按下：準備劃線
wrap.onmousedown = e => {
    if (mode.value !== "line" || e.target !== img) return;
    isDrawingLine = true;
    const r = wrap.getBoundingClientRect();
    sx = e.clientX - r.left;
    sy = e.clientY - r.top;
    
    line.style.display = "block";
    line.style.left = sx + "px";
    line.style.top = sy + "px";
    line.style.width = "0px";
    line.style.transform = "none";
    e.preventDefault();
};

// 滑鼠移動：更新線段與拖曳標籤
window.onmousemove = e => {
    const r = wrap.getBoundingClientRect();
    
    // 處理標籤拖曳
    if (drag) {
        let x = e.clientX - r.left;
        let y = e.clientY - r.top;
        // 限制在畫布內防止拖出邊界
        x = Math.max(0, Math.min(x, r.width));
        y = Math.max(0, Math.min(y, r.height));
        drag.style.left = x + "px";
        drag.style.top = y + "px";
        return;
    }
    
    // 處理劃線預覽
    if (mode.value !== "line" || !isDrawingLine) return;
    const ex = e.clientX - r.left;
    const ey = e.clientY - r.top;
    const dx = ex - sx;
    const dy = ey - sy;
    const len = Math.hypot(dx, dy);
    
    line.style.width = len + "px";
    line.style.transform = `translateY(-1px) rotate(${Math.atan2(dy, dx)}rad)`;
};

// 滑鼠放開：完成劃線並「均勻分散」數字
window.onmouseup = e => {
    drag = null;
    if (mode.value !== "line" || !isDrawingLine) return;
    isDrawingLine = false;
    line.style.display = "none";
    
    const r = wrap.getBoundingClientRect();
    const ex = e.clientX - r.left;
    const ey = e.clientY - r.top;
    
    const c = Math.max(1, parseInt(count.value, 10) || 1);
    
    // 精準直線等分分佈演算法 (從 0% 到 100% 完全均勻平分)
    for (let i = 0; i < c; i++) {
        let t = c === 1 ? 0 : i / (c - 1);
        let posX = sx + (ex - sx) * t;
        let posY = sy + (ey - sy) * t;
        add(posX, posY, n++);
    }
};

// 復原與清除
undo.onclick = () => {
    let m = stack.pop();
    if (m) {
        m.remove();
        n--;
        if (n < (parseInt(start.value, 10) || 1)) {
            n = parseInt(start.value, 10) || 1;
        }
    }
};

clear.onclick = clearAll;

function clearAll() {
    stack.forEach(x => x.remove());
    stack = [];
    n = parseInt(start.value, 10) || 1;
}

// 連動起始編號更動
start.onchange = () => {
    if(stack.length === 0) {
        n = parseInt(start.value, 10) || 1;
    }
};
</script>
</body>
</html>
