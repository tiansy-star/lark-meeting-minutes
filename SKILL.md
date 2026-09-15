---
name: lark-meeting-minutes
description: "从飞书妙记完整逐字稿生成结构化会议纪要，核对关联日程并读取受邀人员；用户明确要求时创建到指定飞书文件夹。当用户提供飞书妙记链接并要求整理、总结或生成会议纪要时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 飞书会议纪要

基于妙记原始逐字稿生成会议纪要。默认模板见 [`TEMPLATE.md`](TEMPLATE.md)。

## 开始前

1. 完整读取同目录下的 [`TEMPLATE.md`](TEMPLATE.md)。用户本次提供其他模板时，改用用户模板。
2. 使用 `lark-cli skills read` 阅读与当前 CLI 版本匹配的领域说明：

```bash
lark-cli skills read lark-shared
lark-cli skills read lark-minutes
lark-cli skills read lark-vc
lark-cli skills read lark-calendar
```

3. 如果用户要求创建飞书文档，再阅读：

```bash
lark-cli skills read lark-doc
lark-cli skills read lark-doc references/lark-doc-create.md
lark-cli skills read lark-doc references/lark-doc-xml.md
lark-cli skills read lark-doc references/style/lark-doc-style.md
lark-cli skills read lark-doc references/style/lark-doc-create-workflow.md
lark-cli skills read lark-doc references/lark-doc-fetch.md
```

所有飞书读取和写入默认使用 `--as user`。

## 工作流

### 1. 读取妙记逐字稿

从 `/minutes/<minute_token>` 链接提取 `minute_token`，读取完整逐字稿：

```bash
lark-cli minutes +detail --as user \
  --minute-tokens <minute_token> \
  --transcript --overwrite \
  --output-dir .meeting-minutes-work
```

使用返回的 `transcript_file`。妙记的 AI 摘要、章节或待办不是最终纪要的事实依据；会议内容必须从逐字稿独立提炼。

如果妙记返回权限不足，停止并请用户联系妙记所有者授权，不要根据标题或其他资料猜测内容。

### 2. 核对关联日程和邀请人员

从逐字稿首行取得会议日期、开始时间和时长，以妙记标题和会议日期搜索日程：

```bash
lark-cli calendar +search-event --as user \
  --query "<妙记标题>" \
  --start <会议日期> --end <次日日期> \
  --page-size 30
```

对每个候选日程依次执行：

```bash
lark-cli calendar +meeting --as user \
  --event-ids <event_id> --calendar-id primary

lark-cli vc +detail --as user --meeting-ids <meeting_id>
```

只有 `vc +detail` 返回的 `minute_token` 与原妙记完全一致，才认定为同一场会议。不得仅凭标题相同采用日程名单。

确认匹配后读取邀请人员：

```bash
lark-cli calendar event.attendees list --as user --params '{
  "calendar_id":"primary",
  "event_id":"<event_id>",
  "page_size":100
}'
```

`has_more=true` 时继续分页。写入 `type=user`、`type=third_party` 和 `type=chat` 的对象；排除会议室等 `resource`。群聊保持为群聊，不自行展开群成员。

无法确认关联日程、没有邀请名单或无日程权限时，省略“参会人员（按日程邀请）”，不得使用逐字稿说话人代替。

### 3. 整理内容

以逐字稿为唯一事实基础，区分：

- **问题**：已经明确存在的规则、流程、字段、责任或协作偏差。
- **结论**：会议已经确认采用、修改或否决的方案。
- **待确认**：仍需业务、开发、财务或其他团队确认的事项。

写作要求：

- 不记录寒暄、投屏故障、态度表述和无实质内容的讨论过程。
- 每个议题只保留影响规则、范围、数据口径、交付或责任分工的信息。
- 关键决策合并同类项，保持 1 至 5 条，只写最终决定。
- 没有明确决策时写“本次未形成需要固化的关键决策”。
- 不补造负责人、截止时间、字段规则或系统行为。
- 无负责人但存在明确行动时写“相关团队”。
- 冲突口径以会议最后明确确认的内容为准；仍不明确的放入待确认事项。

### 4. 套用模板

默认读取 [`TEMPLATE.md`](TEMPLATE.md)。模板中的花括号表示待替换内容，HTML 注释表示生成规则，不要把变量名或注释输出到最终纪要。

用户提供自己的模板文件、飞书文档或明确章节要求时，以用户要求为准，同时保留事实准确性和“已确认/待确认”的区分。

文档标题默认使用：

```text
YYYY-MM-DD 会议纪要：<会议主题>
```

日期取妙记实际录制开始日期，不使用文档创建日期或整理日期。妙记标题为“新录音”等无意义名称时，根据逐字稿核心议题提炼主题。

### 5. 输出或创建飞书文档

- 只提供妙记链接时，在对话中输出纪要，不创建文档。
- 只有用户明确要求创建，并提供目标飞书文件夹时，才执行写入。
- 使用用户指定的目标 token 调用 `docs +create --parent-token`，默认按当前 `lark-doc` 指南使用 XML。
- 创建成功后用 `docs +fetch` 回读，核对文档标题、固定章节、参会人员和所有有内容的条件章节。
- 如果创建结果因网络或超时不确定，先查询结果，不得直接重复创建同名文档。

### 6. 临时文件

逐字稿仅用于本次处理。完成后只清理本次命令创建的 `.meeting-minutes-work` 内容，不得删除用户已有文件。用户要求保留逐字稿时不清理，并告知保存位置。

## 权限不足

如果返回用户权限不足，按报错中的缺失 scope 增量发起授权。若返回飞书开放平台 `console_url`，将链接原样提供给用户或组织管理员；不要声称 AI 可以代替管理员审批。
