# 部署指南：Supabase + EdgeOne Pages

本项目为 **AI 助教工作台** 的单文件前端应用（`index.html`），通过 Supabase 提供真实的用户注册与登录功能，并托管于腾讯 EdgeOne Pages。

---

## 目录

1. [项目结构](#项目结构)
2. [需要填写的配置信息](#需要填写的配置信息)
3. [Supabase 配置步骤](#supabase-配置步骤)
4. [EdgeOne Pages 部署步骤](#edgeone-pages-部署步骤)
5. [功能说明](#功能说明)
6. [常见问题](#常见问题)

---

## 项目结构

```
my-edgeone-supabase-app/
├── index.html   # 整个应用（前端 + Supabase Auth 集成）
└── SETUP.md     # 本文档
```

---

## 需要填写的配置信息

打开 `index.html`，找到顶部的两行配置（约第 448–449 行），替换为你的实际值：

```javascript
const SUPABASE_URL      = 'https://YOUR_PROJECT_ID.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

| 变量名 | 说明 | 获取位置 |
|---|---|---|
| `SUPABASE_URL` | 项目 API 地址 | Supabase 控制台 → Project Settings → API → **Project URL** |
| `SUPABASE_ANON_KEY` | 匿名公钥 | 同上页面 → **anon** `public` 一栏 |

> **注意**：`anon` key 是公开密钥，可以安全地放在前端代码中。不要使用 `service_role` key。

---

## Supabase 配置步骤

### Step 1 — 注册并创建项目

1. 访问 <https://supabase.com>，注册或登录账号
2. 点击 **New project**
3. 填写以下信息：

   | 字段 | 说明 |
   |---|---|
   | Organization | 选择或新建一个组织 |
   | Project name | 任意，例如 `ai-tutor` |
   | Database password | 设置一个强密码并妥善保存 |
   | Region | 选择 **Southeast Asia (Singapore)** 以降低延迟 |

4. 点击 **Create new project**，等待约 1 分钟完成初始化

---

### Step 2 — 关闭邮件验证（必须执行）

本项目使用 `用户名@user.local` 格式的虚拟邮箱注册，**必须关闭邮件确认**，否则注册后无法直接登录。

1. 左侧菜单 → **Authentication** → **Providers**
2. 找到 **Email** 一栏，点击进入
3. 关闭 **Confirm email** 开关
4. 点击 **Save** 保存

---

### Step 3 — 获取 API 密钥

1. 左侧菜单 → **Project Settings** → **API**
2. 记录以下两个值：

   | 值 | 对应配置变量 |
   |---|---|
   | Project URL | `SUPABASE_URL` |
   | `anon` `public` Key | `SUPABASE_ANON_KEY` |

---

### Step 4 — 将密钥填入代码

打开 `index.html`，将第 448–449 行替换为你的实际值，例如：

```javascript
const SUPABASE_URL      = 'https://abcdefghijkl.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx';
```

---

## EdgeOne Pages 部署步骤

### Step 1 — 将代码推送到 GitHub

```bash
cd my-edgeone-supabase-app

# 初始化 Git 仓库
git init
git add index.html
git commit -m "feat: connect supabase auth"

# 在 GitHub 新建一个仓库后，关联并推送
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin main
```

---

### Step 2 — 在腾讯云开通 EdgeOne Pages

1. 登录 [腾讯云控制台](https://console.cloud.tencent.com)
2. 搜索 **EdgeOne**，进入 **边缘安全加速平台 EdgeOne**
3. 左侧菜单选择 **Pages**
4. 点击 **创建项目** → **从 Git 导入**

---

### Step 3 — 授权并选择仓库

1. 点击 **连接 GitHub**，完成 OAuth 授权
2. 在仓库列表中找到并选择刚才推送的仓库
3. 点击 **开始设置**

---

### Step 4 — 配置构建参数

| 配置项 | 填写内容 |
|---|---|
| 项目名称 | 任意，例如 `ai-tutor` |
| 生产分支 | `main` |
| 构建命令 | （留空） |
| 构建输出目录 | `.`（英文句点，代表根目录） |
| 根目录 | `/`（默认即可） |

---

### Step 5 — 部署

1. 点击 **保存并部署**，等待约 30 秒
2. 部署完成后 EdgeOne 会分配一个 `*.edgeone.app` 的访问域名
3. 打开该域名即可使用应用

---

### Step 6 — 配置 Supabase 允许的域名（可选但推荐）

为了防止其他网站滥用你的 Supabase 项目，建议添加生产域名白名单：

1. Supabase 控制台 → **Authentication** → **URL Configuration**
2. 在 **Site URL** 中填写 EdgeOne 分配的域名，例如 `https://ai-tutor.edgeone.app`
3. 在 **Redirect URLs** 中同样添加该域名
4. 点击 **Save**

---

## 功能说明

| 功能 | 实现方式 |
|---|---|
| 用户注册 | Supabase Auth `signUp`，user_metadata 存储用户名、角色等信息 |
| 用户登录 | Supabase Auth `signInWithPassword` |
| 保持登录 | Supabase 自动管理 Session，刷新页面不掉线 |
| 退出登录 | Supabase Auth `signOut` |
| 角色区分 | 注册时通过 `user_metadata.role` 字段标记（`teacher` / `student`） |
| 其余 UI 功能 | 使用内置 Mock 数据演示，不依赖后端 |

### 注册时的用户名规则

- 用户名只能包含字母、数字和下划线
- 长度至少 3 个字符
- 系统会在内部将其转换为 `用户名@user.local` 格式存储到 Supabase

---

## 常见问题

**Q：注册后提示"请前往 Supabase 控制台关闭 Confirm email"**

A：执行 [Step 2](#step-2--关闭邮件验证必须执行) 中的操作，关闭邮件验证后重新注册。

---

**Q：登录时提示 "Invalid login credentials"**

A：可能原因：
- 该用户名尚未注册，请先切换到「注册」Tab
- 密码输入有误

---

**Q：部署到 EdgeOne 后页面空白**

A：打开浏览器开发者工具（F12）查看 Console 报错。最常见的原因是 `SUPABASE_URL` 或 `SUPABASE_ANON_KEY` 仍为占位符未替换。

---

**Q：如何在 Supabase 查看已注册的用户？**

A：Supabase 控制台 → **Authentication** → **Users**，可以看到所有注册用户的列表。
