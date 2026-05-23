# outlook

一个基于 `ruyipage + Firefox` 的 Outlook / Hotmail 注册工具。  
当前版本已经改成**启动时不再交互提问**，而是**只读取 `config.json` 运行**。

---

## 1. 环境要求

- Windows x64
- Python 3.11+ 或通过 `uv` 自动安装 Python
- Firefox 浏览器运行时

> 仓库**不再内置 Firefox 二进制**，因为浏览器运行时体积较大，不适合直接放进普通 Git 仓库。

---

## 2. 使用 uv 安装环境

### 2.1 安装 uv

官方 Windows PowerShell 安装方式：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安装完成后，重新打开一个 PowerShell，检查是否可用：

```powershell
uv --version
```

如果你更喜欢 `winget`，也可以：

```powershell
winget install --id=astral-sh.uv -e
```

### 2.2 安装 Python（可选）

如果系统还没有合适的 Python，可以直接让 `uv` 安装：

```powershell
uv python install 3.12
```

### 2.3 创建虚拟环境并安装依赖

在项目目录下执行：

```powershell
uv venv --python 3.12
.venv\Scripts\activate
uv pip install -r requirements.txt
```

如果你已经有可用 Python，也可以把 `3.12` 换成你本机的版本。

---

## 3. 下载 Firefox 浏览器

### 方案 A：直接安装系统 Firefox（最省事，推荐）

从 Mozilla 官方下载页下载安装：

- Firefox for Windows: https://www.mozilla.org/en-US/firefox/windows/

安装完成后，常见路径是：

```text
C:\Program Files\Mozilla Firefox\firefox.exe
```

然后把 `config.json` 里的 `ruyipage.browser_path` 改成这个路径，例如：

```json
"ruyipage": {
    "browser_path": "C:/Program Files/Mozilla Firefox/firefox.exe",
    "profile_root": "Profiles",
    "headless": false,
    "xpath_picker": false,
    "action_visual": false
}
```

### 方案 B：使用你自己的独立 Firefox 运行时

如果你不想安装到系统目录，也可以自己准备一个 Firefox 目录，只要最终能拿到：

```text
...\firefox\firefox.exe
```

然后把 `browser_path` 指向那个 `firefox.exe` 即可。

---

## 4. 配置文件

### 4.1 复制模板

先把模板复制成实际运行配置：

```powershell
Copy-Item .\config.example.json .\config.json
```

### 4.2 修改 `config.json`

当前模板已经带了公开可用的 `client_id`：

```json
"client_id": "1fdedcda-5469-456b-a58b-6d65f557e5e0"
```

你至少要确认这些字段：

- `proxy_source`
- `proxy_file` 或 `proxy_api_url`
- `concurrent_flows`
- `max_tasks`
- `oauth2.enable_oauth2`
- `oauth2.client_id`
- `ruyipage.browser_path`

示例：

```json
{
    "choose_browser": "ruyipage",
    "email_suffix": "@outlook.com",
    "proxy_source": "freefile",
    "proxy_file": "proxies.txt",
    "proxy_api_url": "",
    "proxy_api_timeout": 8,
    "proxy_test_urls": [
        "https://outlook.live.com/mail/0/?prompt=create_account",
        "https://login.live.com"
    ],
    "proxy_test_timeout": 8,
    "bot_protection_wait": 11,
    "max_captcha_retries": 2,
    "concurrent_flows": 5,
    "max_tasks": 20,
    "show_logs": true,
    "oauth2": {
        "enable_oauth2": true,
        "client_id": "1fdedcda-5469-456b-a58b-6d65f557e5e0",
        "redirect_url": "http://localhost:8000",
        "Scopes": [
            "offline_access",
            "https://graph.microsoft.com/Mail.ReadWrite",
            "https://graph.microsoft.com/Mail.Send",
            "https://graph.microsoft.com/User.Read"
        ]
    },
    "ruyipage": {
        "browser_path": "C:/Program Files/Mozilla Firefox/firefox.exe",
        "profile_root": "Profiles",
        "headless": false,
        "xpath_picker": false,
        "action_visual": false
    }
}
```

---

## 5. 代理配置

如果使用文件代理：

### `freefile`

`proxies.txt` 每行一个：

```text
HOST:PORT
```

例如：

```text
127.0.0.1:10809
127.0.0.1:10809
127.0.0.1:10809
```

> 当前代码已支持：**同一个代理重复写多行，就按多并发槽位处理**。

### `file`

每行一个：

```text
HOST:PORT:USER:PASS
```

### `api`

把 `config.json` 里的：

```json
"proxy_source": "api",
"proxy_api_url": "你的接口地址"
```

改好即可。

---

## 6. 运行

激活虚拟环境后直接运行：

```powershell
python .\main.py
```

现在程序会：

1. 直接读取 `config.json`
2. 自动启动并发任务
3. 不再弹出启动时的人工输入菜单

---

## 7. 输出结果

运行结果默认会写到 `Results/` 目录：

- `Results\unlogged_email.txt`  
  仅注册成功但未做 OAuth2 授权的账号密码

- `Results\logged_email.txt`  
  注册成功并启用了 OAuth2 的账号密码

- `Results\outlook_token.txt`  
  保存 OAuth2 刷新令牌，格式为：

```text
邮箱----密码----client_id----refresh_token
```

---

## 8. 常见问题

### 8.1 为什么提示找不到 Firefox？

通常是 `config.json` 里的：

```json
"browser_path"
```

没有填对。

请确认它指向真实存在的：

```text
firefox.exe
```

并建议在 JSON 里使用：

- `C:/Program Files/...`
- 或 `C:\\Program Files\\...`

不要写成非法 JSON 路径。

### 8.2 为什么只跑一个线程？

先检查两件事：

1. `concurrent_flows` 是否大于 1
2. `proxies.txt` 是否真的提供了足够的并发槽位

当前版本已经支持：

- 多个不同代理并发
- 同一个代理重复多行并发复用

### 8.3 为什么没有生成 refresh token？

要满足下面几个条件：

- `oauth2.enable_oauth2 = true`
- `client_id` 正确
- 授权流程成功

成功后会写入：

```text
Results\outlook_token.txt
```

---

## 9. 仓库说明

以下内容默认不提交到 Git：

- `config.json`
- `proxies.txt`
- `Results/`
- `Profiles/`
- `.venv/`
- 本地 Firefox 运行时

也就是说，仓库里保留的是：

- 代码
- 示例配置
- 使用说明

本地运行态文件请自行准备。
