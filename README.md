# 會員積分管理系統 (Loyalty System)

這是一個輕量級、基於 Web 的單一檔案 (Single-File) 會員積分管理應用程式。它整合了 **Supabase** 作為後端資料庫與認證服務，並使用 **Tailwind CSS** 進行快速的響應式介面設計。系統區分為「用戶端」與「店家管理端」。

## 🛠 技術架構

- **前端核心**: 原生 HTML5, JavaScript (ES6+)
    
- **樣式框架**: [Tailwind CSS](https://tailwindcss.com/ "null") (CDN)
    
- **後端服務**: [Supabase](https://supabase.com/ "null") (Auth, Database, Realtime) - 使用 `@supabase/supabase-js` v2
    
- **圖標庫**: [Phosphor Icons](https://phosphoricons.com/ "null")
    
- **部署形式**: Serverless / 靜態網頁 (無需 Node.js 後端伺服器)
    

## ✨ 功能特色

### 👤 用戶端 (User Client)

- **註冊/登入**: 支援 Email 與密碼註冊，自動生成會員顯示編號 (Display ID)。
    
- **帳號救援**: 註冊時自動生成「助記詞 (Mnemonic)」，用於忘記密碼時重設帳號。
    
- **個人儀表板**: 檢視目前積分餘額、會員基本資料 (姓名、電話、編號)。
    
- **響應式設計**: 針對手機移動端優化的介面。
    

### 🏪 店家管理端 (Admin Dashboard)

- **權限識別**: 系統自動識別 Admin 角色並切換至管理介面。
    
- **會員列表**: 檢視所有會員清單，顯示姓名、電話與目前積分。
    
- **即時搜尋**: 支援依姓名或電話快速篩選會員。
    
- **積分管理**:
    
    - **彈窗操作**: 點擊會員進行管理。
        
    - **交易紀錄**: 支援「增加」或「扣除」點數，並需填寫交易備註 (Reason)。
        
    - **資料修改**: 可直接修改會員的姓名與電話。
        

## 🚀 安裝與設定指南

由於本專案為單一 HTML 檔案，部署非常容易，但必須先設定 Supabase 後端。

### 1. Supabase 資料庫設定

請在 Supabase 專案的 SQL Editor 中執行以下指令以建立必要的資料表與函式：

```
-- 1. 建立 Profiles 資料表 (延伸 Auth 資料)
create table public.profiles (
  id uuid references auth.users not null primary key,
  email text,
  full_name text,
  phone text,
  user_id_display text,
  mnemonic text, -- 注意：生產環境建議加密儲存
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

-- 3. 啟用 RLS (Row Level Security) - 根據需求設定 Policy (此處略過詳細 Policy 設定)
alter table public.profiles enable row level security;
alter table public.point_logs enable row level security;

-- 4. 建立「透過助記詞重設密碼」的 RPC 函式 (Postgres Function)
-- 注意：此功能需要較高權限來修改 auth.users
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
  -- 驗證 Email 和 助記詞 是否匹配
  select id into target_user_id
  from public.profiles
  where email = target_email and mnemonic = mnemonic_check;

  if target_user_id is not null then
    -- 更新 auth.users 的密碼 (Supabase 內部表)
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

### 2. 應用程式設定

1. 打開 `index.html` 檔案。
    
2. 找到 `<script>` 區塊頂部的 **CONFIG** 區域。
    
3. 填入您的 Supabase 專案資訊：
    

```
// ==========================================
// CONFIG
// ==========================================
const SUPABASE_URL = '[https://your-project-id.supabase.co](https://your-project-id.supabase.co)'; 
const SUPABASE_ANON_KEY = 'your-anon-key-here';
// ==========================================
```

### 3. 啟動專案

- 直接使用瀏覽器打開 `index.html` 即可運行。
    
- 或將檔案上傳至任何靜態網頁託管服務 (如 Vercel, Netlify, GitHub Pages)。
    

## 📝 使用說明

1. **管理員設定**: 註冊一個新帳號後，進入 Supabase 資料庫介面，手動將該用戶在 `profiles` 資料表中的 `role` 欄位修改為 `admin`。
    
2. **一般用戶**: 註冊後會看到紅色警告視窗顯示「助記詞」，請務必提醒用戶截圖或抄寫，這是忘記密碼時的唯一救援手段。
    

## ⚠️ 注意事項

- **安全性**: 本範例將助記詞 (`mnemonic`) 明文儲存於 `profiles` 表中以便比對，生產環境建議進行雜湊 (Hash) 處理或加密。
    
- **瀏覽器支援**: 需支援 ES6 JavaScript 的現代瀏覽器。
    
- **Supabase Client**: 程式碼中使用 `window.supabase` 進行全域呼叫，請確保 CDN 載入正常。
    

## 📄 授權

此專案僅供內部使用或教學參考。

```

### 檔案說明 (給用戶參考)：
這份 README 結構清晰，包含了**技術細節**、**功能列表**以及最重要的**資料庫 Schema (SQL)**。由於您的程式碼依賴特定的資料表結構和 RPC 函式才能運作，這份文件能幫助任何接手的人快速建立環境。
```