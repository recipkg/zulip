# 来源与未完成项

- 上游 [zulip/zulip](https://github.com/zulip/zulip)，提交 `dd5adc2de6c43bd7bac7d1a0e6dc18c9364268a7`，README 标注 Apache 2.0。
- 阅读 README、`zerver/models/messages.py`、`zerver/openapi/zulip.yaml` 中消息查询部分和上游帮助/API 索引；尚未审阅完整代码、测试与 OpenAPI。
- 从固定提交的 OpenAPI、`zerver/lib/events.py` 注册流程和 `zerver/tornado/event_queue.py` 队列实现补充了实时事件与确认合同，见 `EVENT-SYNC-SOURCE.md`；这是源码对照，尚未运行服务或验证竞争与重连。
- 尚未运行上游服务或客户端，也没有独立重建。未完成：组织/频道/私聊权限、实时事件、全量 API 与错误、同步和未读、搜索、消息格式、通知、附件、部署及备份、国际化、性能和真实设备视觉验收。
- 质量和设计合同是固定提交的配方依赖；外部服务或源代码仓库不是配方依赖。
