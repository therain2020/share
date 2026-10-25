# Repo-Map Skill 架构表述方案

编排 Skill + 双行为分支架构的三个表述版本，按场合取用。

## 一、中文标准版（技术方案 / 需求文档）

整体采用编排Skill + 双行为分支的协同架构：

1. **编排Skill（Orchestrator Skill）**：作为总控入口，负责整条Skill调度链路，并根据用户意图进行分支决策——判断当前任务走「查询行为分支」还是「更新行为分支」；
2. **查询行为分支**：
   - 拓扑查询工具Skill（Repo-Map Query Skill）：读取 `repo-map.md`，返回当前任务涉及的关联仓库依赖清单；
   - 上下文检索工具Skill（Context Retrieval Skill）：基于拓扑结果比对当前工作区，检索缺失的跨仓库代码定义，信息不全时主动向用户提问，引导补充对应仓库；
3. **更新行为分支**：
   - 拓扑更新工具Skill（Repo-Map Update Skill）：根据用户提供的新增仓库、依赖变更或服务拆分信息，更新 `repo-map.md` 中的仓库依赖拓扑，保持拓扑表与实际架构同步。

四层均为纯提示词规则实现，无 MCP 外部工具调用，适配公司禁用 MCP 的管控策略。

## 二、极简口语版（开会 / 口头同步）

一个编排Skill做总控，判断用户是要查还是要更：查就走拓扑查询+上下文检索，缺代码就主动问补仓库；更就走拓扑更新，直接改repo-map。

## 三、英文汇报版（外企 / global 同步）

We adopt an Orchestrator + dual-branch Skill architecture:

1. **Orchestrator Skill**: Central controller that routes each request to either the Query branch or the Update branch based on user intent;
2. **Query Branch**:
   - Repo-Map Query Tool Skill: Reads repo-map.md and returns dependency relationships for the involved repositories;
   - Context Retrieval Tool Skill: Validates loaded repos against the topology, detects missing cross-repo definitions, and proactively prompts users to add related repositories;
3. **Update Branch**:
   - Repo-Map Update Tool Skill: Updates repo-map.md based on new repos, dependency changes, or service splits, keeping the topology in sync with the actual architecture.

All layers are prompt-based with no MCP dependencies, compliant with internal MCP restrictions.

## 四、架构图标签

```plaintext
编排Skill（Orchestrator）
├── 分支A：查询行为
│   ├── 拓扑查询工具Skill（repo-map.md 读）
│   └── 上下文检索工具Skill
└── 分支B：更新行为
    └── 拓扑更新工具Skill（repo-map.md 写）
```

## 五、分支决策逻辑（一句话）

编排Skill解析用户意图 → 意图为「分析/编码/查依赖」走查询分支 → 意图为「新增仓库/改依赖/同步架构」走更新分支 → 分支执行完毕后由编排Skill统一收口回复。

## 六、分支判定规则示例（可写进编排Skill）

| 用户意图关键词 | 路由分支 |
| --- | --- |
| 这个接口在哪、为什么报错、帮我写代码、缺什么上下文 | 查询分支 |
| 新增了 xxx 服务、repo 依赖变了、更新拓扑、加个仓库 | 更新分支 |
| 意图不明确 | 编排Skill主动追问：「你是要查询跨仓库上下文，还是更新 repo-map 拓扑？」 |
