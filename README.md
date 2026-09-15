# 飞书会议纪要 Skill

把飞书妙记链接交给 AI，自动生成结构化会议纪要；需要时还可以读取日程邀请人员，并把纪要创建到指定飞书文件夹或知识库。

## 最简单的安装方法

把这个仓库的地址发给 Codex 或其他支持 Agent Skills 的 AI，然后发送：

```text
请安装这个会议纪要 Skill。完整阅读仓库中的 INSTALL_FOR_AI.md，帮我完成飞书 CLI 安装、Skill 安装、飞书授权和安装验证。
```

AI 会自动执行安装。你只需要在出现提示时：

1. 打开 AI 提供的飞书配置或授权链接；
2. 确认授权；
3. 如果组织要求审批，请管理员批准应用权限。

AI 无法绕过飞书管理员审批，也不应该替你确认授权。

## 如何使用

### 只生成会议纪要

```text
使用 $lark-meeting-minutes，根据这条飞书妙记生成会议纪要：

<妙记链接>
```

### 生成并创建到飞书目录

```text
使用 $lark-meeting-minutes，根据妙记生成会议纪要，并创建到指定飞书目录。

妙记：<妙记链接>
目录：<飞书文件夹或知识库目录链接>
```

只提供妙记链接时，AI 默认在对话中输出纪要，不会擅自创建飞书文档。

### 使用自己的模板

默认模板是 [`TEMPLATE.md`](TEMPLATE.md)。你可以直接修改安装后的这个文件，也可以在每次使用时提供其他模板：

```text
使用 $lark-meeting-minutes 生成会议纪要，并使用这个模板：
<模板文件路径或飞书文档链接>

妙记：<妙记链接>
```

## 默认纪要内容

- 会议信息；
- 会议摘要；
- 核心讨论内容；
- 关键决策；
- 待办事项（有明确行动时）；
- 风险与待确认问题（存在未决事项时）；
- 后续安排（有明确安排时）。

默认文档名称：

```text
YYYY-MM-DD 会议纪要：<会议主题>
```

## 需要的飞书权限

安装时 AI 会一次性发起相关权限申请。给组织管理员说明时，可以使用下面这张表：

| 权限范围 | 用途 |
| --- | --- |
| 妙记 | 读取妙记基本信息和完整逐字稿 |
| 日历 | 搜索关联日程并读取邀请人员 |
| 视频会议 | 核对日程会议与妙记是否属于同一场会议 |
| 云文档 | 创建会议纪要并在创建后回读检查 |
| 云空间 | 识别目标文件夹和文档信息 |
| 知识库 | 将纪要创建到指定知识库目录 |

对应的主要用户权限为：

```text
offline_access
minutes:minutes.basic:read
minutes:minutes.artifacts:read
calendar:calendar.event:read
vc:meeting.meetingevent:read
vc:record:readonly
docx:document:create
docs:document.content:read
drive:drive.metadata:readonly
wiki:wiki:readonly
wiki:node:read
wiki:node:create
```

不同飞书 CLI 版本可能为同一接口提供兼容 scope。安装时以 CLI 返回的缺失权限和飞书开放平台审批页为准。

## 使用前提

- 你对妙记有阅读权限；
- 需要邀请名单时，你能读取对应日程；
- 需要创建文档时，你对目标目录有写入权限；
- 妙记逐字稿已经生成完成。

## 隐私说明

AI 需要临时下载逐字稿才能整理纪要。Skill 默认只把逐字稿用于本次任务，完成后清理本次生成的临时文件。不要把包含敏感会议内容的临时文件提交到代码仓库或分享给无关人员。

## 手动安装

如果不通过 AI，可以执行：

```bash
npx @larksuite/cli@latest install
npx skills add . -y -g
lark-cli config init
lark-cli auth login --domain minutes --domain calendar --domain vc --domain docs --domain drive --domain wiki
lark-cli auth status --json --verify
```

安装后重新打开 Codex；在输入框中输入 `$lark-meeting-minutes` 检查 Skill 是否可用。

## 常见问题

### 无权读取妙记

请让妙记所有者向你的飞书账号开放阅读权限，并确认 CLI 登录账号正确。

### 找不到日程邀请人员

即时会议可能没有关联日程。无法通过妙记 Token 确认关联关系时，Skill 会省略邀请人员，不会用逐字稿说话人冒充日程名单。

### 没有创建飞书文档

必须明确要求创建，并提供有写入权限的飞书文件夹或知识库目录链接。

### Skill 没有出现

重新打开 Codex。仍未出现时，让 AI 检查 `lark-meeting-minutes/SKILL.md` 是否已安装到个人 Skills 目录。

