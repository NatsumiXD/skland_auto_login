# Skland-Sign-In

森空岛自动签到脚本，用于《明日方舟》和《终末地》的每日签到。

本项目已改造为 **GitHub Actions 定时运行**，无需 Docker、无需服务器常驻。

## 功能特性

- 支持单账号和多账号签到
- 支持 GitHub Secrets 管理 Token
- 支持手动触发和定时触发
- 支持多种通知渠道（Qmsg / OneBot / 邮件 / 企业微信 / 微信服务号 / Server酱）

## 1. 如何获取 Token

1. 登录森空岛官网：https://www.skland.com/
2. 打开：https://web-api.skland.com/account/info/hg
3. 复制返回 JSON 中 `content` 字段的完整字符串，作为签到 Token

示例：

```json
{"code":0,"data":{"content":"这里是一长串Token"}}
```

## 2. GitHub Actions 使用

工作流文件：`.github/workflows/sign-in.yml`

### 一键创建自己的仓库

可以在 README 放按钮，让用户一键跳转：

[![Fork](https://img.shields.io/badge/GitHub-Fork-blue?logo=github)](https://github.com/OWNER/REPO/fork)
[![Use this template](https://img.shields.io/badge/GitHub-Use%20this%20template-success?logo=github)](https://github.com/new?template_name=REPO&template_owner=OWNER)

使用说明：

- `OWNER` 和 `REPO` 需要替换为你的实际仓库地址。
- 如果仓库启用了 Template（仓库设置中勾选 `Template repository`），建议优先使用 `Use this template`。
- GitHub 暂不支持通过按钮自动写入 Actions Secrets，所以用户仍需在自己仓库手动配置 `SKLAND_TOKEN(S)` 或 `SKLAND_USERS_JSON`。

已配置：

- `workflow_dispatch`：手动执行
- `schedule`：定时执行（默认 UTC `0 17 * * *`，约等于北京时间每日 01:00）

## 3. 配置仓库 Secrets

路径：`Settings -> Secrets and variables -> Actions -> New repository secret`

支持以下三种方式，脚本按优先级读取：

1. `SKLAND_USERS_JSON`（优先级最高）
2. `SKLAND_TOKENS`
3. `SKLAND_TOKEN`

### 方式 A：单账号

Secret 名称：`SKLAND_TOKEN`

值：

```text
你的Token
```

### 方式 B：多账号（简单）

Secret 名称：`SKLAND_TOKENS`

值可用换行或逗号分隔：

```text
token_1
token_2
```

或

```text
token_1,token_2
```

### 方式 C：多账号 + 自定义昵称（推荐）

Secret 名称：`SKLAND_USERS_JSON`

值：

```json
[
  {"nickname": "大号", "token": "token_1"},
  {"nickname": "小号", "token": "token_2"}
]
```

## 4. 运行方式

### 手动运行

1. 打开仓库 `Actions`
2. 选择 `Skland Sign In`
3. 点击 `Run workflow`

### 定时运行

默认每天自动执行一次，无需额外操作。

如果要修改时间，编辑 `.github/workflows/sign-in.yml` 里的 `cron`：

```yaml
schedule:
  - cron: "0 17 * * *"
```

说明：GitHub Actions 使用 UTC 时区。

## 5. 本地运行（可选）

如果你希望本地测试：

1. 安装依赖

```bash
pip install -r requirements.txt
```

2. 配置账号（二选一）

- 使用环境变量：`SKLAND_TOKEN` / `SKLAND_TOKENS` / `SKLAND_USERS_JSON`
- 或复制 `config.example.yaml` 为 `config.yaml`，填写 `users`

3. 执行

```bash
python main.py
```

## 6. 通知配置（可选）

通知配置在 `config.yaml` 的 `notify` 节点中，按需填写即可启用。

支持渠道：

- Qmsg
- OneBot V11
- 邮件 SMTP
- 企业微信机器人
- 微信服务号模板消息
- Server酱

## 7. 常见问题

### 1) 工作流运行了但没有签到

请先检查：

- Secret 是否正确配置
- Token 是否有效（是否过期）
- Actions 日志中是否有账号解析失败信息

### 2) 多账号没有按预期显示昵称

请使用 `SKLAND_USERS_JSON`，并确保 JSON 格式合法。

## 致谢

本项目核心 API 交互逻辑（`skland_api.py`）提取自 AstrBot 开源插件：
https://github.com/Azincc/astrbot_plugin_skland
