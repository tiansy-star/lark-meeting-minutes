# AI 安装指南

本文件面向负责部署的 AI Agent。目标是让用户以尽可能少的操作安装 `lark-meeting-minutes`，完成飞书配置、授权和验证。

## 安全边界

- 可以自动执行只读检查和常规安装命令。
- 不得输出或记录 App Secret、access token 等敏感凭证。
- 配置和 OAuth 链接必须原样交给用户，并同时生成二维码。
- 组织管理员审批和用户授权必须由对应人员完成，AI 不得声称可以代为批准。
- 安装阶段不得创建测试飞书文档，除非用户明确同意并提供目标目录。
- 不要使用 `--domain all`；只申请本工作流需要的业务域。

## 1. 检查环境

检查：

```bash
node --version
npm --version
command -v lark-cli
lark-cli --version
```

如果没有 Node.js，停止并请用户先安装 Node.js 当前长期支持版本。

如果没有 `lark-cli`，或版本明显过旧，执行：

```bash
npx @larksuite/cli@latest install
```

已有 CLI 时可以先检查更新：

```bash
lark-cli update --check
```

不要未经用户同意强制覆盖非标准或源码安装版本。

## 2. 安装 Skill

如果当前已经位于本仓库根目录，执行：

```bash
npx skills add . -y -g
```

如果用户只给了远程仓库地址，使用安装器支持的远程仓库形式安装：

```bash
npx skills add <repository-url> -y -g
```

确认安装结果中存在 `lark-meeting-minutes/SKILL.md` 和 `TEMPLATE.md`。不要在多个个人 Skill 目录重复安装同名 Skill。

如果当前 Codex 任务不能立刻发现新 Skill，告诉用户安装已完成，并建议新建任务或重新启动 Codex。

## 3. 检查飞书配置

执行：

```bash
lark-cli config show
```

如果尚未配置，运行：

```bash
lark-cli config init --new
```

该命令可能阻塞等待用户操作。使用可持续读取输出的后台会话运行。取得配置 URL 后：

1. 不得修改 URL；
2. 使用 `lark-cli auth qrcode` 生成 PNG 二维码；
3. 把原始链接和二维码一起展示给用户；
4. 等待用户在浏览器中完成配置；
5. 继续读取原命令直到成功或明确失败。

如果飞书要求组织管理员审批，把审批说明和 CLI 返回的 `console_url` 原样交给用户。等待审批完成后再继续。

## 4. 检查并发起用户授权

先检查现有登录：

```bash
lark-cli auth status --json --verify
```

如果尚未登录、Token 失效或权限不完整，使用一次性业务域授权：

```bash
lark-cli auth login \
  --domain minutes \
  --domain calendar \
  --domain vc \
  --domain docs \
  --domain drive \
  --domain wiki \
  --no-wait --json
```

从本次返回值中取得 `verification_url` 和 `device_code`。随后：

1. 将 `verification_url` 原样展示给用户；
2. 用以下命令生成二维码：

```bash
lark-cli auth qrcode '<verification_url>' --output lark-auth.png
```

3. 展示二维码；
4. 告诉用户：“请完成授权后回来告诉我已授权，我会继续完成安装。”
5. 结束当前轮次，不要在用户看见链接前阻塞轮询。

用户确认完成后，使用本次授权尝试返回的 device code：

```bash
lark-cli auth login --device-code <device_code>
```

device code 只能用于对应的当前授权尝试。过期、失败或上下文丢失时，重新发起授权，不得复用旧 code。

## 5. 验证权限

执行：

```bash
lark-cli auth status --json --verify
```

应确认：

- 当前有效身份为 `user`；
- 用户 Token 有效；
- 用户确实是预期登录账号；
- 妙记、日历、视频会议、文档、云空间和知识库相关权限已授权。

可以检查核心 scopes：

```bash
lark-cli auth check --json --scope "offline_access minutes:minutes.basic:read minutes:minutes.artifacts:read calendar:calendar.event:read vc:meeting.meetingevent:read vc:record:readonly docx:document:create docs:document.content:read drive:drive.metadata:readonly wiki:wiki:readonly wiki:node:read wiki:node:create"
```

若 CLI 返回的缺失权限属于兼容 scope 或名称发生变化，读取当前版本说明：

```bash
lark-cli skills read lark-shared
lark-cli skills read lark-minutes
lark-cli skills read lark-calendar
lark-cli skills read lark-vc
lark-cli skills read lark-doc
```

根据明确的缺失 scope 增量授权。不要扩大到无关业务域。

## 6. 完成安装

不要使用虚构的妙记链接进行网络测试。完成后告诉用户：

1. 飞书 CLI 版本；
2. Skill 安装位置；
3. 登录和核心权限状态；
4. 是否仍等待管理员审批；
5. 首次使用方式。

建议给用户下面的提示词：

```text
使用 $lark-meeting-minutes，根据这条飞书妙记生成会议纪要：

<妙记链接>
```

如果用户随后提供真实妙记，再按 `SKILL.md` 执行。只有用户明确要求创建文档并提供目标目录时，才写入飞书。
