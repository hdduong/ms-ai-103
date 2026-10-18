# 06 Use group chat orchestration

[Use group chat orchestration](https://learn.microsoft.com/en-us/training/modules/orchestrate-semantic-kernel-multi-agent-solution/6-use-group-chat-orchestration?pivots=text)

- Agents share one thread. A chat manager chooses who speaks and when a person should join. Agents talk. They do not change live systems.
- Use it for brainstorming, a debate, maker-checker (one writes, one reviews, repeat), or a review you can audit because it is one transcript.
- Skip it for a straight pipeline, when you need a fast answer, or with more than about three agents. The manager has to know when to stop.
- `GroupChatBuilder`. A custom `GroupChatManager` can change the rules.
- Each round, in order: `should_request_user_input`, `should_terminate`, then either `filter_results` or `select_next_agent`.
