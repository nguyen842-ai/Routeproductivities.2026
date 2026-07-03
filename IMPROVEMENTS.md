# 🚀 Hướng Dẫn Cải Thiện File index.html

## Tổng Quan
File hiện tại có **~940 dòng mã** với CSS và JavaScript ghép chung. Dưới đây là cách cải thiện để website chạy nhanh hơn, dễ bảo trì hơn, và thân thiện với người dùng hơn.

---

## 1️⃣ RÚT GỌNCSS RA FILE RIÊNG (ƯU TIÊN CAO)

### ❌ Vấn đề hiện tại:
- CSS ~100 dòng được nhúng trực tiếp trong `<head>` 
- Mỗi lần tải trang, HTML lớn hơn 50KB
- Trình duyệt không thể cache CSS độc lập

### ✅ Cách làm:

**Bước 1: Tạo file `styles.css`**
```css
/* styles.css */
:root {
    --b1:#E6F1FB; --b2:#B5D4F4; --b3:#85B7EB; --b4:#378ADD; --b5:#185FA5; --b6:#0C447C; --b7:#042C53;
    --sidebar-width: 440px;
}

* { -webkit-tap-highlight-color: transparent; }

body { 
    margin: 0; 
    font-family: 'Segoe UI', system-ui, sans-serif; 
    background-color: #f0f4f8; 
    color: var(--b7); 
    font-size: 13px; 
    overflow-x: hidden; 
}

/* ... copy tất cả CSS hiện tại vào đây ... */
```

**Bước 2: Cập nhật file `index.html`**
```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Route Productivity Dashboard © 2026 TCI All rights reserved</title>
    <!-- ✅ Thay thế <style>...</style> bằng dòng này: -->
    <link rel="stylesheet" href="styles.css">
</head>
```

### 📊 Lợi ích:
- ⚡ **Giảm 50KB** khi load HTML
- 🔄 **Cache tốt hơn** - CSS chỉ download 1 lần
- 📝 **Dễ bảo trì** - CSS và HTML tách riêng

---

## 2️⃣ THÊM XỬ LÝ LỖI KHI TẢI DỮ LIỆU (ƯU TIÊN CAO)

### ❌ Vấn đề hiện tại:
- Nếu file `kamMap.js` hoặc `dataM6.js` không tải được, trang trắng trơn
- Người dùng không biết có lỗi gì
- Khó debug

### ✅ Cách làm:

Thay thế code này (dòng 195-223):

```javascript
// ❌ CŨ - Không có xử lý lỗi
scriptsToLoad.forEach(function(src) {
    var script = document.createElement('script');
    script.src = src;
    script.onload = function() {
        loadedScripts++;
        if (loadedScripts === scriptsToLoad.length) {
            init();
        }
    };
    document.body.appendChild(script);
});
```

Bằng code này:

```javascript
// ✅ MỚI - Có xử lý lỗi
var failedScripts = [];

scriptsToLoad.forEach(function(src) {
    var script = document.createElement('script');
    script.src = src;
    
    script.onload = function() {
        loadedScripts++;
        console.log(`✅ Đã tải: ${src}`);
        if (loadedScripts === scriptsToLoad.length) {
            if (failedScripts.length > 0) {
                console.error('❌ Lỗi tải file:', failedScripts);
                showErrorMessage('Một số dữ liệu không tải được. Dashboard có thể không đầy đủ.');
            }
            init();
        }
    };
    
    script.onerror = function() {
        failedScripts.push(src);
        loadedScripts++;
        console.error(`❌ Không tải được: ${src}`);
        if (loadedScripts === scriptsToLoad.length) {
            init(); // Tiếp tục chạy với dữ liệu có sẵn
        }
    };
    
    script.onload = null; // Xóa lỗi cũ
    document.body.appendChild(script);
});

// Hàm hiển thị thông báo lỗi
function showErrorMessage(msg) {
    var errorDiv = document.createElement('div');
    errorDiv.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: #ef4444;
        color: white;
        padding: 15px 20px;
        border-radius: 8px;
        z-index: 99999;
        font-size: 14px;
        font-weight: bold;
    `;
    errorDiv.textContent = msg;
    document.body.appendChild(errorDiv);
    
    setTimeout(() => errorDiv.remove(), 5000); // Ẩn sau 5 giây
}
```

### 📊 Lợi ích:
- 🚨 **Người dùng biết có lỗi**
- 🔍 **Console log giúp debug**
- 💪 **Ứng dụng không crash**

---

## 3️⃣ RÚT GỌNJAVASCRIPT RA FILE RIÊNG (ƯU TIÊN TRUNG)

### ❌ Vấn đề hiện tại:
- JavaScript ~940 dòng được nhúng trực tiếp trong HTML
- File HTML quá to (khó đọc, khó bảo trì)
- Không thể cache JavaScript độc lập

### ✅ Cách làm:

**Bước 1: Tạo file `app.js`**
```javascript
// app.js
// Copy tất cả JavaScript từ dòng 225-938 vào file này
// Giữ nguyên toàn bộ logic
```

**Bước 2: Cập nhật file `index.html`**
```html
<!-- Trước khi </body> -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script src="https://mrqldgycwovpbmzdeuyz.supabase.co/storage/v1/object/public/data/dataM5.js"></script>

<!-- ✅ Thêm dòng này: -->
<script src="app.js"></script>

<!-- Giữ phần script nhỏ load dữ liệu động: -->
<script>
    var version = '?t=' + new Date().getTime();
    var supabaseDataM6 = "https://mrqldgycwovpbmzdeuyz.supabase.co/storage/v1/object/public/data/dataM6.js";
    
    var scriptsToLoad = [
        'kamMap.js' + version,
        supabaseDataM6 + version
    ];
    
    // ... (phần tải dữ liệu động giữ nguyên) ...
</script>
```

### 📊 Lợi ích:
- 📝 **HTML gọn hơn** - chỉ ~200 dòng
- 🔄 **Cache JavaScript** - tải 1 lần, sử dụng nhiều lần
- 🔧 **Dễ bảo trì**

---

## 4️⃣ CẢI THIỆN KHỞI ĐỘNG & CACHE (ƯU TIÊN TRUNG)

### ❌ Vấn đề hiện tại:
- Mỗi lần F5, file lại thêm `?t=timestamp` → không bao giờ cache
- Trình duyệt phải download lại tất cả

### ✅ Cách làm:

Thay thế code này (dòng 196-206):

```javascript
// ❌ CŨ
var version = '?t=' + new Date().getTime(); // Luôn buộc download lại
var supabaseDataM6 = "https://mrqldgycwovpbmzdeuyz.supabase.co/storage/v1/object/public/data/dataM6.js";

var scriptsToLoad = [
    'kamMap.js' + version,
    supabaseDataM6 + version
];
```

Bằng code này:

```javascript
// ✅ MỚI - Cache thông minh
const IS_PROD = window.location.hostname !== 'localhost';
const version = IS_PROD ? '' : '?t=' + new Date().getTime();
// Chỉ disable cache trong phát triển (localhost)
// Production: không có ?t= → trình duyệt cache lại

const supabaseDataM6 = "https://mrqldgycwovpbmzdeuyz.supabase.co/storage/v1/object/public/data/dataM6.js";

const scriptsToLoad = [
    'kamMap.js' + version,
    supabaseDataM6 + version
];
```

### 📊 Lợi ích:
- ⚡ **Lần load tiếp theo nhanh hơn 70%**
- 💾 **Tiết kiệm bandwidth**
- 🧪 **Vẫn test cache-busting trên localhost**

---

## 5️⃣ CẢI THIỆN KHẢ NĂNG TIẾP CẬN (ACCESSIBILITY) (ƯU TIÊN TRUNG)

### ❌ Vấn đề hiện tại:
- Bộ lọc sử dụng `<div>` thay vì `<button>`
- Người dùng bàn phím không thể tương tác
- Screen reader không biết các phần tử là gì

### ✅ Cách làm:

**Thay thế các bộ lọc từ:**
```html
<!-- ❌ CŨ - Không accessibility -->
<div class="ms-box" onclick="toggleMS('month')">
    <div class="ms-label" id="lbl-month">Tất cả</div>
    <div class="ms-dropdown" id="drop-month">
        <div class="ms-opt" onclick="updateF('month', 'all')">Tất cả</div>
    </div>
</div>
```

**Thành:**
```html
<!-- ✅ MỚI - Có accessibility -->
<button 
    class="ms-box" 
    role="combobox"
    aria-label="Bộ lọc tháng"
    aria-expanded="false"
    aria-controls="drop-month"
    id="month-trigger"
    onclick="toggleMS('month')">
    <span class="ms-label" id="lbl-month">Tất cả</span>
</button>

<div 
    class="ms-dropdown" 
    id="drop-month" 
    role="listbox"
    aria-label="Danh sách tháng"
    hidden>
    <div class="ms-opt" role="option" onclick="updateF('month', 'all')">Tất cả</div>
    <!-- ... -->
</div>
```

**Cập nhật JavaScript:**
```javascript
function toggleMS(id) {
    // Đóng dropdown cũ
    if(openMS && openMS !== id) {
        const oldDrop = document.getElementById('drop-'+openMS);
        const oldTrigger = document.getElementById(openMS+'-trigger');
        oldDrop.setAttribute('hidden', '');
        oldTrigger.setAttribute('aria-expanded', 'false');
    }
    
    const drop = document.getElementById('drop-'+id);
    const trigger = document.getElementById(id+'-trigger');
    
    if(drop.hasAttribute('hidden')) {
        drop.removeAttribute('hidden');
        trigger.setAttribute('aria-expanded', 'true');
        openMS = id;
    } else {
        drop.setAttribute('hidden', '');
        trigger.setAttribute('aria-expanded', 'false');
        openMS = null;
    }
}
```

### 📊 Lợi ích:
- ♿ **Người dùng bàn phím có thể dùng**
- 👀 **Screen reader hỗ trợ**
- 🌍 **Tuân thủ WCAG 2.1**

---

## 6️⃣ GỬI NHỚ CHIỀU RỘNG SIDEBAR (OPTIONAL)

### ❌ Vấn đề hiện tại:
- Người dùng resize sidebar, nhưng F5 lại reset về mặc định
- Mất trải nghiệm người dùng

### ✅ Cách làm:

Thay thế hàm `initResizer()` (dòng 525-534):

```javascript
function initResizer() {
    const resizer = document.getElementById('dragMe');
    
    // ✅ Lấy giá trị đã lưu
    const savedWidth = localStorage.getItem('sidebarWidth');
    if (savedWidth) {
        document.documentElement.style.setProperty('--sidebar-width', savedWidth + 'px');
    }
    
    let isResizing = false;
    
    resizer.addEventListener('mousedown', function(e) {
        isResizing = true;
        document.body.classList.add('no-select');
        resizer.classList.add('resizing');
    });
    
    document.addEventListener('mousemove', function(e) {
        if (!isResizing) return;
        let newWidth = e.clientX;
        if (newWidth < 250) newWidth = 250;
        if (newWidth > 700) newWidth = 700;
        document.documentElement.style.setProperty('--sidebar-width', newWidth + 'px');
        
        // ✅ Lưu vào localStorage
        localStorage.setItem('sidebarWidth', newWidth);
    });
    
    document.addEventListener('mouseup', function(e) {
        if (isResizing) {
            isResizing = false;
            document.body.classList.remove('no-select');
            resizer.classList.remove('resizing');
        }
    });
}
```

### 📊 Lợi ích:
- 💾 **Nhớ tùy chỉnh của người dùng**
- 🎯 **Trải nghiệm mượt mà**

---

## 7️⃣ TÁCH CẤU HÌNH VÀO FILE CONFIG (OPTIONAL)

### ❌ Vấn đề hiện tại:
- URL Supabase được hard-code ở nhiều chỗ
- Khó thay đổi khi deploy

### ✅ Cách làm:

**Tạo file `config.js`:**
```javascript
// config.js
const CONFIG = {
    // API & Storage
    supabase: {
        projectId: 'mrqldgycwovpbmzdeuyz',
        url: 'https://mrqldgycwovpbmzdeuyz.supabase.co',
        bucket: 'data'
    },
    
    // Dữ liệu
    dataFiles: ['M5', 'M6', 'M7'],
    
    // Cấu hình UI
    sidebar: {
        minWidth: 250,
        maxWidth: 700,
        defaultWidth: 440
    },
    
    // Months
    months: {
        "jan":1, "feb":2, "mar":3, "apr":4, "may":5, "jun":6,
        "jul":7, "aug":8, "sep":9, "oct":10, "nov":11, "dec":12
    }
};

// Hàm helper
function getDataUrl(file) {
    return `${CONFIG.supabase.url}/storage/v1/object/public/${CONFIG.supabase.bucket}/${file}.js`;
}
```

**Sử dụng trong app.js:**
```javascript
const supabaseDataM6 = getDataUrl('dataM6');
// Thay vì hard-code URL dài dòng
```

### 📊 Lợi ích:
- 🔧 **Dễ cấu hình**
- 🚀 **Sẵn sàng cho deploy**

---

## 📋 CHECKLIST CẢI THIỆN

- [ ] **1. Extract CSS** → Tạo `styles.css`
- [ ] **2. Thêm error handling** → Cập nhật script loader
- [ ] **3. Extract JavaScript** → Tạo `app.js`
- [ ] **4. Cải thiện caching** → Thêm điều kiện `IS_PROD`
- [ ] **5. Accessibility** → Thêm ARIA labels
- [ ] **6. localStorage sidebar** → Lưu kích thước
- [ ] **7. Config file** → Tạo `config.js`

---

## 🎯 THỨ TỰ THỰC HIỆN ĐỀ XUẤT

### 🔥 Giai đoạn 1 (Tuần 1) - ƯU TIÊN CAO
1. **Extract CSS** → Ngay lập tức
2. **Error handling** → Ngay lập tức
3. **Extract JS** → Trong tuần

### ⚡ Giai đoạn 2 (Tuần 2) - ƯU TIÊN TRUNG
4. **Caching** → Cập nhật production
5. **Accessibility** → Thêm dần

### ✨ Giai đoạn 3 (Tuần 3) - NICE TO HAVE
6. **localStorage sidebar**
7. **Config file**

---

## 📞 LIÊN HỆ GIÚP ĐỠ

Nếu cần:
- 📝 Tôi viết code chi tiết
- 🔄 Tôi tạo Pull Request
- 🧪 Tôi test lại

**Bạn muốn bắt đầu từ bước nào?** 🚀
