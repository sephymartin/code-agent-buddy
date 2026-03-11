# code-agent-buddy

这是一个面向 AI 编码助手的规范与技能仓库，当前主要用于沉淀 Java 后端开发相关的写码约束、数据库建模规则，以及 MyBatis-Plus 持久层实践。

仓库本身不是业务应用，也不是可独立运行的服务。它的实际作用是为 Codex、Cursor 一类工具提供可复用的 `skills` 和规则文件，帮助代理在生成代码、设计表结构、评审持久层实现时更贴近项目约定。

## 仓库当前包含什么

目前仓库主要分为两类内容：

1. `skills/`
用于定义可被代理调用的技能说明，告诉代理在特定任务下应该遵循什么约束、输出什么结构。

当前已有技能：

- `java-code-writing`
  - 面向通用 Java / Spring 后端编码
  - 约束点包括：Lombok 使用、构造注入、事务边界、工具类写法、同步服务边界等
- `mybatis-plus`
  - 面向 MyBatis-Plus 持久层开发与评审
  - 约束点包括：`*DO` 实体建模、`*Dao` 约定、分页、批量操作、局部更新安全性等
- `database-design`
  - 约束点包括：主键与外键命名、字段类型选择、索引设计、长文本拆表、冗余业务编号、DO 映射说明等

2. `.cursor/rules/`
用于定义 Cursor 规则，约束代理在特定文件类型下的输出风格与实现方式。

当前已有规则：

- `database-standards.mdc`
  - SQL 建表规范
- `java/java-code-style.mdc`
  - Java 编码风格与分层约束
- `java/mybatis-plus.mdc`
  - MyBatis-Plus 实体、Dao、分页、批量与更新规范
- `mise-config.mdc`
  - 要求代理在处理项目任务前优先检查 `mise` 配置

## 仓库适合解决的问题

这个仓库当前更适合下面几类场景：

- 让代理按统一风格生成 Java / Spring 后端代码
- 让代理按 MyBatis-Plus 约定实现 Dao、DO、分页和批量操作
- 让代理输出贴合现有规范的数据库设计方案
- 让 Cursor/Codex 在代码生成和评审时遵守统一规则，而不是给出过于通用的建议

## 使用方式

### 1. 在 Codex 中使用技能

如果运行环境支持技能发现与调用，可以直接让代理使用对应技能，例如：

- 使用 `java-code-writing` 生成普通 Java 服务代码
- 使用 `mybatis-plus` 设计或评审持久层代码
- 使用 `database-design` 输出数据库设计章节

其中 `database-design/agents/openai.yaml` 已声明该技能可被隐式调用，并给出了默认提示语。

### 2. 在 Cursor 中使用规则

将 `.cursor/rules/` 下的规则文件纳入工程后，代理在处理匹配文件时会更倾向遵守这些约束。例如：

- 写 `*.java` 文件时遵循 Java 代码风格与分层规范
- 写 `*Dao.java`、`*DO.java` 时遵循 MyBatis-Plus 规则
- 写 `*.sql` 时遵循建表命名、字段、索引和注释规范

## 推荐外部 Skills

除了当前仓库内置的领域技能外，也推荐搭配外部技能仓库 `superpowers` 使用。

`superpowers` 主要提供代理工作流能力，例如需求澄清、方案设计、计划编写、系统化调试、完成前验证和代码评审流程，适合与本仓库的 Java / MyBatis-Plus / 数据库建模规范组合使用。

项目链接：

- [obra/superpowers](https://github.com/obra/superpowers)

## 当前仓库特征

基于当前代码内容，这个仓库有几个明显特征：

- 聚焦 Java 后端，不包含前端、测试框架或部署脚本类技能
- 强调规范沉淀，而不是代码运行
- 同时兼顾通用技能说明和 Cursor 规则两种载体

## 维护建议

如果后续继续扩展这个仓库，建议按下面的方向保持一致：

- 新技能继续放在 `skills/<skill-name>/SKILL.md`
- 与技能强绑定的参考资料放在对应目录下的 `references/`
- 针对特定代理平台的适配配置放在 `agents/`
- 通用编辑器/代理规则继续放在 `.cursor/rules/`
- README 只描述仓库当前真实存在的能力，避免写入尚未落地的功能

## 结论

`code-agent-buddy` 当前可以理解为一个“面向 AI 编码代理的 Java 后端规范知识库”。它已经沉淀了 Java 编码、MyBatis-Plus 持久层、以及后端数据库建模三类核心能力，适合作为团队统一代理行为和生成质量的基础仓库。
