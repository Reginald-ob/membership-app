# 🏸 會員積分管理系統 (SPA Pro Version)

這是一個基於 Single Page Application (SPA) 架構的輕量級會員積分管理系統。整合 Supabase 作為後端，並使用 Tailwind CSS 進行介面設計。

## 💡 為什麼選擇 SPA (單頁應用) 架構？
為了提供如原生 App 般的流暢體驗，本專案刻意維持「單一 HTML 檔案」的結構。

- 避免「假性登出」：在傳統多頁面網頁中，跳轉頁面（如從首頁跳到商店頁）會導致瀏覽器重新整理，使用者視覺上會經歷「畫面閃爍」或「重新載入」，容易產生「我是不是被登出了？」的錯覺。
- 狀態即時性：SPA 架構下，使用者的登入狀態（currentUser）與資料（如點數餘額）均暫存於記憶體中，切換分頁（如從「我的」切換到「優惠」）時無需重新向伺服器請求，反應速度極快。

---

## 🛠 關鍵程式碼與功能對照指南

以下列出專案中核心的程式碼區塊，說明修改它們會如何影響網站的運作與數值。

### 1. 核心參數設定 (Config)
位於 `<script>` 標籤最頂端，控制全域邏輯。

🔹 點數回饋比例 (`CAT_RATES`) 控制不同消費類別的點數回饋 % 數。

```javascript
const CAT_RATES = { 
    'A': 0.04, // A類 (臨租): 修改 0.04 -> 0.10 即變為 10% 回饋
    'B': 0.01, // B類 (月租): 修改 0.01 -> 0.02 即變為 2% 回饋
    'C': 0.03, // C類 (裝備)
    'D': 0.00, // D類 (穿線)
    'E': 0.05  // E類 (消耗品)
};
```

影響：店家端在輸入消費金額時，系統自動計算「預計獲得點數」的倍率。

🔹 點數發行上限 (`MAX_SUPPLY`) 控制「點數發行概況」卡片中的進度條分母。

```javascript
const MAX_SUPPLY = 500000; // 修改此數字可調整發行總量上限
```

影響：若改為 `1000000`，且目前已發行量不變，則進度條的百分比會減半。

---

### 2. 介面文字與顯示 (UI/UX)
🔹 點數發行概況卡片 (HTML) 位於 `tab-info` 區塊內，控制卡片顯示的靜態文字與動態數據容器。

```html
<!-- HTML 結構 -->
<div class="stat-row">
    <span class="stat-label">發行上限 (Max Supply)</span>
    <!-- id="stat-max-supply" 用於 JS 注入數據 -->
    <span class="stat-value" id="stat-max-supply">500,000</span>
</div>
<div class="stat-row">
    <span class="stat-label">已發行 (Circulating)</span>
    <!-- id="stat-circulating" 會由 updateCirculationStats() 自動更新 -->
    <span class="stat-value" id="stat-circulating">計算中...</span>
</div>
```

修改文字：直接修改 `<span class="stat-label">...</span>` 內的文字即可更改標籤名稱。

🔹 點數發行計算邏輯 (JS) 位於 `updateCirculationStats()` 函式。

```javascript
async function updateCirculationStats() {
    // ... 從資料庫獲取所有用戶點數 ...
    
    // 計算百分比邏輯
    // 若要改變顯示格式 (例如顯示小數點後 2 位)，請修改 .toFixed(1) 為 .toFixed(2)
    const percentage = Math.min(100, (totalIssued / MAX_SUPPLY) * 100).toFixed(1);
    
    // ... 更新 UI ...
}
```

🔹 側邊選單內容 (Sidebar) 位於 `<div id="user-sidebar-panel">` 內。

```html
<!-- Menu Item -->
<div>
    <a href="#" class="...">
        選單項目一  <!-- 修改此處文字可變更選單名稱 -->
    </a>
</div>
```

---

### 3. 店家管理端邏輯 (Admin Logic)

🔹 點數計算與扣抵 (`calculatePoints`)

當店家輸入金額或扣抵點數時觸發此函式。

```javascript
window.calculatePoints = function() {
    // ... 獲取輸入值 ...

    // 計算邏輯：(消費金額 - 扣抵點數) * 類別匯率
    // 若要取消「扣抵部分不回饋」的邏輯，可將 effectiveAmount 改為 amount
    const effectiveAmount = Math.max(0, amount - deductPoints);
    const earnedPoints = Math.floor(effectiveAmount * rate);

    // ... UI 顯示變動 ...
}
```

---

### 4. 樣式修改指南 (CSS)
主要樣式使用 Tailwind CSS（Utility classes），部分客製化樣式位於 `<style>` 區塊。

🔹 Uiverse 卡片樣式 (`.notification`)

```css
.notification {
    /* 卡片背景色 */
    background: #29292c; 
    /* 卡片寬度限制，修改 max-width 可改變卡片在桌面版的大小 */
    max-width: 20rem; 
    /* 邊框流光動畫的顏色設定 */
    --gradient: linear-gradient(to bottom, #2eadff, #3d83ff, #7e61ff);
    --color: #32a6ff;
}
```

🔹 底部導覽列 (`.nav-item`)

```css
/* 被選中的按鈕顏色 */
.nav-item.active { color: #4f46e5; } 
/* 被選中時圖標放大的比例 */
.nav-item.active i { transform: scale(1.1); } 
```

---

## 🚀 資料庫安裝 (Supabase SQL)
請在 Supabase 的 SQL Editor 執行以下指令，以建立完整的資料表結構與 RPC 函式：

```sql
-- 1. 建立 Profiles 資料表
create table public.profiles (
  id uuid references auth.users not null primary key,
  email text,
  full_name text,
  phone text,
  user_id_display text,
  mnemonic text,
  points int4 default 0,
  role text default 'user', -- 'admin' 或 'user'
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- 2. 建立積分交易紀錄表
create table public.point_logs (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id),
  admin_id uuid references public.profiles(id),
  change_amount int4 not null,
  reason text,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- 3. 安全性設定 (RLS)
alter table public.profiles enable row level security;
alter table public.point_logs enable row level security;

-- 簡易 Policy (開發測試用，正式上線建議設定更嚴格)
create policy "Public profiles are viewable by everyone" on public.profiles for select using (true);
create policy "Users can update own profile" on public.profiles for update using (auth.uid() = id);
create policy "Logs viewable by everyone" on public.point_logs for select using (true);
create policy "Logs insertable by everyone" on public.point_logs for insert with check (true);

-- 4. 建立重設密碼函式 (RPC)
create or replace function reset_password_via_mnemonic(
  target_email text,
  mnemonic_check text,
  new_password text
)
returns boolean
language plpgsql
security definer
as $$
declare
  target_user_id uuid;
begin
  select id into target_user_id
  from public.profiles
  where email = target_email and mnemonic = mnemonic_check;

  if target_user_id is not null then
    update auth.users
    set encrypted_password = crypt(new_password, gen_salt('bf'))
    where id = target_user_id;
    return true;
  else
    return false;
  end if;
end;
$$;
```

---

## ⚠️ 注意事項
- 管理員權限：註冊帳號後，需手動進入 Supabase 資料庫，將該用戶的 `role` 欄位改為 `admin`，重新登入後即可進入後台。  
- Supabase URL/Key：請務必在 `index.html` 中的 CONFIG 區塊填入您自己的專案資訊。

---

### 📋 總結
這份 README 清楚地解釋了：
1. **架構選擇**：為什麼堅持不拆分 HTML 頁面 (SPA)。  
2. **商業邏輯**：點數匯率（`CAT_RATES`）和發行上限（`MAX_SUPPLY`）在哪裡改。  
3. **UI 邏輯**：介面文字和 CSS 樣式如何調整。  
4. **後端邏輯**：資料庫 Schema 的 SQL 語法。