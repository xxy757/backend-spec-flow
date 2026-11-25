# OpenSpec 执行方式

以下接口已按本机 OpenSpec 1.4.1 验证。实际运行先检查 `openspec --version` 和相关 `--help`；不将其他版本文档中的参数直接套用。优先采用 CLI 输出中的路径，不假设当前目录就是 OpenSpec 根目录。项目使用外部规划目录或 workspace 时遵守返回的编辑范围；无法定位代码目标时先澄清。

## 定位及接入

1. 阅读项目指令，定位实际 OpenSpec 项目。在该目录执行 `openspec list --json`；有多个 change 时根据明确上下文选取，仍不明确再请用户选择。
2. 对已有 change 执行 `openspec status --change "<change>" --json`，沿用其 schema，不通过覆盖参数假装迁移。
3. 对用户已要求使用本流程的新 change：将本 Skill 的 `assets/backend-diagram-flow/` 复制到目标项目的 `openspec/schemas/backend-diagram-flow/`。已有同名 schema 时先比较，不覆盖用户修改；存在实质冲突时保留原文件，解决选用问题后再继续。
4. 执行 `openspec schema validate backend-diagram-flow --json`。成功后用 `openspec new change "<change>" --schema backend-diagram-flow` 创建本次 change。不修改 `openspec/config.yaml` 的默认 schema，也不改旧 change。
5. 项目没有 OpenSpec 或工具不可用时，说明缺项；仅在用户授权范围内初始化或安装。不得把另一个目录的项目误当成本次目标。纯讨论和审核不接入 schema。

## 规划循环

运行 `openspec status --change "<change>" --json`，读取 artifact 的状态与 requires、applyRequires 及实际路径。

对依赖已满足的产物运行：

```sh
openspec instructions <artifact-id> --change "<change>" --json
```

先读取返回的 context、rules、instruction、template 和 dependencies，再按返回的输出位置生成。context 和 rules 用于约束写作，不原样复制成规格；将本次具体行为和设计决策落实到相应文档。产物完成后重新检查 status。

本 schema 顺序为 proposal → design → specs → tasks，design 是必需产物。图和 Spec 可以在确认前迭代，但任务拆分不能先于二者。沿用其他 schema 时按其真实依赖顺序；若设计可选，本 Skill 仍要求本次业务图和数据图有实际阅读入口，不得借“可选”省略图。

检查 applyRequires 及其传递依赖都实际存在且内容完整。`done` 只表明文件存在，不意味着设计正确或已经确认。模板中的提示注释应在生成真实产物时替换为内容或删除。

在 design.md 中维护设计版本和真实确认来源。需要用户确定时展示图与 Spec 入口及未决问题；确认规则见 SKILL.md。用户若只要求设计，完成设计后停止，不开始实现。

## 实现与恢复

```sh
openspec status --change "<change>" --json
openspec instructions apply --change "<change>" --json
```

- 使用 CLI 返回的 contextFiles 读取最新文件。Mermaid 图被链接到外部 Markdown 或 .mmd 时同时读其权威源。
- `blocked` 时解决返回的缺项，不直接实现；`ready` 时仍需核对设计确认记录；`all_done` 只反映任务标记，不能代替验收。
- 从剩余任务恢复，逐项落实；只在任务描述的工作实际完成后勾选。测试任务失败、跳过或被阻塞时保持未完成并记录原因。
- 出现需求或设计差异时按主 Skill 更新图、Spec 和确认范围，不能静默加机制或缩减约定行为。

## 验收

在 change 的 verification.md 写入需求／场景、图路径、代码符号、实际测试结果的对应关系，参考 worked-example.md 的样式。验证本次行为和图的显示，再执行：

```sh
openspec validate "<change>" --strict --no-interactive
```

按实际版本调整命令。该命令不验证 Mermaid 渲染、用户确认的真实性、代码行为或无额外设计；这些必须分别审查。Skill 格式检查和 schema 校验也不能替代实际任务演练。

## 限制与维护

此 schema 是独立的轻量工作流，不修改 OpenSpec 安装包。现阶段 apply.requires 明列 proposal、design、specs、tasks，可以让 CLI 在缺文件时报告 blocked；确认记录和语义一致性仍由 agent 与用户检查，不宣称有自动语义门禁。

schema 与模板一起维护。升级 OpenSpec 后在临时项目重新验证依赖顺序、模板读取、缺项时 blocked 和正常任务跟踪，再应用到新 change。
