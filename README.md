# WeirdHost 自动登录与服务器时间管理

本项目用于自动登录 WeirdHost / Pterodactyl 面板，并执行服务器时间增加任务，同时支持截图管理和 Telegram 通知。适合需要长期自动运行的场景。

## 功能

1. **自动登录服务器页面**  
   - 支持 **Cookie 登录** 或 **邮箱密码登录**  
   - 优先使用 Cookie 登录，失败时回退到邮箱密码登录  

2. **自动点击 "시간 추가" 按钮**  
   - 脚本会自动查找目标服务器页面的按钮并点击  
   - 点击成功或失败都会在日志和 Telegram 中显示  

3. **截图管理**  
   - 每次运行都会截图失败或操作页面  
   - 自动删除 **15 天前** 的旧截图  
   - 保留 **最新 50 张** 截图  

4. **Telegram 通知**  
   - 登录失败 / 按钮点击失败 / 其他异常都会推送通知  
   - 显示登录方式（Cookie 或邮箱密码）和具体错误  
   - 支持中文提示  

---

## 登录方式

本项目支持 **两种登录方式**：

### 1️⃣ 使用 Cookie 登录（推荐）

**适用场景：**  
- 保持会话，无需每次输入账号密码  
- 避免频繁登录，适合长期自动运行的任务  

**操作步骤：**  
1. 登录目标网站（如 Pterodactyl 面板）  
2. 打开浏览器开发者工具（F12 或右键选择“检查”）  
3. 找到 **Application → Cookies**  
4. 找到与你登录相关的 Cookie（如 `remember_web_59ba36addc2b2f9401580f014c7f58ea4e30989d` 或 session、auth_token）  
5. 复制该 Cookie 值  
6. 在 GitHub 仓库页面，进入 **Settings → Secrets and variables → Actions → New repository secret**  

```
名称: REMEMBER_WEB_COOKIE
值: 复制的 cookie
```

### 2️⃣ 使用邮箱和密码登录

**适用场景：**  
- 没有有效 Cookie  
- 或希望使用账号密码登录  

**操作步骤：**  
1. 在 GitHub 仓库页面，进入 **Settings → Secrets and variables → Actions → New repository secret**  
2. 添加两个 Secrets：  

```
名称: PTERODACTYL_EMAIL
值: 你的 Pterodactyl 账户邮箱

名称: PTERODACTYL_PASSWORD
值: 你的 Pterodactyl 账户密码
```

**注意：**  
- 同时提供 Cookie 和邮箱密码时，脚本会优先使用 Cookie  
- Cookie 登录失败会回退到邮箱密码登录  

---

## 登录逻辑总结

1. 脚本首先检查是否有 `REMEMBER_WEB_COOKIE`  
   - 有则使用 Cookie 登录  
   - 登录成功 → 直接进入服务器页面  
   - 登录失败 → 清除 Cookie，回退到邮箱密码登录  

2. 如果没有 Cookie，脚本会使用 `PTERODACTYL_EMAIL` 和 `PTERODACTYL_PASSWORD` 登录  

3. 登录和操作结果会在 **Telegram 通知**中显示，告知登录方式及是否成功  

---

## Telegram 通知配置

需要在 GitHub Secrets 中添加：

```
TG_BOT_TOKEN  # Telegram 机器人 token
TG_CHAT_ID    # 你的聊天 ID
```

通知内容会用中文显示，例如：

- ✅ 成功点击按钮  
- ❌ 登录失败（Cookie / 邮箱密码）  
- ❌ 按钮点击失败  

---

## GitHub Actions 自动运行

- 使用 `.github/workflows/add_time.yml` 配置  
- 每天自动运行（可修改 cron 表达式）  
- 自动执行以下步骤：  
  1. 登录服务器页面  
  2. 点击 "시간 추가" 按钮  
  3. 保存截图（失败时）  
  4. 清理旧截图（15 天前）  
  5. 保留最新 50 张截图  
  6. 提交更新截图到仓库  
