# 实时事件与客户端同步：固定源码合同

本页依据 Zulip 固定提交 `dd5adc2de6c43bd7bac7d1a0e6dc18c9364268a7` 的 [OpenAPI](https://github.com/zulip/zulip/blob/dd5adc2de6c43bd7bac7d1a0e6dc18c9364268a7/zerver/openapi/zulip.yaml)、[注册实现](https://github.com/zulip/zulip/blob/dd5adc2de6c43bd7bac7d1a0e6dc18c9364268a7/zerver/lib/events.py) 和 [事件队列实现](https://github.com/zulip/zulip/blob/dd5adc2de6c43bd7bac7d1a0e6dc18c9364268a7/zerver/tornado/event_queue.py) 提炼。它是源码和接口定义证据，不是运行结果，也不是完整同步协议。

## 注册和获取

- 已认证用户通过 `POST /api/v1/register` 注册队列并获取所请求的初始状态；返回 `queue_id` 与初始 `last_event_id`。`event_types` 筛选未来入队事件，省略时接收所有支持的类型；`fetch_event_types` 筛选初始状态，省略则使用 `event_types`，两者均省略则不按类型筛选。服务器忽略不支持的类型。`queue_id` 对公开访问的未认证请求可为 `null`；下面的工作流仅针对已认证用户。
- 固定源码先分配队列、获取初始状态，再把获取状态期间排队的事件应用到响应状态，并返回所应用事件的最后 ID；没有事件时为 `-1`。实现兼容该流程时应保留初始化期间变化，避免快照与后续队列之间出现丢失。不同类型是否纳入初始数据取决于筛选条件；不能假设它是所有数据的完整快照。
- 以 `GET /api/v1/events` 提供 `queue_id` 和上次已处理的 `last_event_id`。返回的 `events` 按队列事件 ID 递增，但 ID 不保证连续；事件 ID 是队列游标，不是消息 ID，也不应跨队列比较。`dont_block=true` 时无事件可返回空数组；默认长轮询直至新事件或心跳。客户端超时采用注册响应的 `event_queue_longpoll_timeout_seconds`，避免因心跳间隔变化提前放弃请求。
- 客户端应先可靠处理响应中的事件，再在下次请求确认最后处理的事件 ID；收到响应本身不等于确认。固定源码在确认时按 ID 清除队列中不大于该 ID 的事件，并检查确认点；未确认前以同一游标重复请求可用于恢复，但确认后不能把已清除的旧游标当作可回放历史。应持久化业务处理结果及游标的关联，避免客户端崩溃时重复副作用或漏处理；仅靠服务器队列不承诺端到端恰好一次。
- 队列归属用户：固定源码用用户 ID 校验，错误用户或不存在的队列均返回 `BAD_EVENT_QUEUE_ID`，不能仅凭获知队列 ID 读取他人事件。队列删除或闲置被回收后，读取该队列得到 HTTP 400、`result=error`、`code=BAD_EVENT_QUEUE_ID`；客户端重新注册并重新建立状态，而不是继续使用旧游标。`DELETE /api/v1/events` 以表单 `queue_id` 删除已注册队列。
- `idle_queue_timeout` 请求可指定秒数或 `mobile`，实际值由响应的 `idle_queue_timeout_secs` 给出；接口定义列出默认 10 分钟和上限 7 天。心跳和清理时间受服务器配置及调度影响，不以单次等待证明准确回收时间。

## 筛选、消息和已知边界

- 注册的 `narrow` 过滤收到的消息事件，不自动订阅频道：例如按频道过滤时，默认仍需订阅该频道才收到其新消息；`all_public_streams` 等选项另有语义，不能笼统声称能接收所有可访问频道消息。
- `message` 事件携带队列事件 `id` 及消息对象。发送私聊 `POST /api/v1/messages` 使用 `type=direct`、`to` 为用户 ID 或 API 邮箱组成的 JSON 数组，`content` 为消息文本；频道消息使用 `type=channel` 或 `stream` 并提供频道和主题。收件人和未授权用户的可见性必须按实例配置和测试场景验证。
- 源码对某些消息标记事件合并以减少队列数据，并在注释中指出读取/未读交错情况下的顺序缺陷；不要把队列建模为每次用户操作必定对应一个独立事件，也不要据此推断所有未读行为已得到验证。
- 各事件类型和字段、多个同时轮询、重启后的队列恢复、权限变更、初始化竞争、不同客户端能力、实际认证方式、并发及异常恢复均未逐项运行或完整提取。必须保留这些未知项。
