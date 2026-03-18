# Skill Creation Best Practices

本文档总结了本仓库在创建和演进 skill 过程中的实践结论，重点覆盖 skill 触发方式、渐进式加载、文档拆分、自检与验证。

## 1. 先定义目标，不要先合并内容

创建或改造 skill 时，先回答一个问题：

- 目标是提升发现率
- 目标是降低上下文消耗
- 目标是减少 skill 数量
- 目标是补充非显然规则

如果目标是节省上下文，不要一开始就把多个 skill 内容直接合并到一个大 `SKILL.md`。更好的做法是保留单一入口，再把低频、重细节的内容拆到 `references/` 中按需读取。

## 2. `description` 只写触发条件，不写 workflow

`SKILL.md` frontmatter 中的 `description` 应只回答一件事：

- 什么时候应该触发这个 skill

不要在 `description` 里解释 skill 内部流程、加载顺序或实现机制。因为代理很可能只看 `description` 就做决定，过度描述 workflow 会让它跳过正文。

推荐写法：

```yaml
description: Use when writing or reviewing Java backend code, especially when tasks involve Spring services or controllers, MyBatis-Plus DO or DAO patterns, paging or batch persistence logic, or repository-aligned schema design.
```

不推荐写法：

```yaml
description: Covers Java and Spring backend code writing, and progressively loads MyBatis-Plus persistence and database conventions...
```

原因：

- 前者描述触发条件
- 后者描述 skill 做什么以及如何做

## 3. 主 `SKILL.md` 保持轻量

主 `SKILL.md` 应只保留高频、低分歧、最值得默认进入上下文的内容，例如：

- skill 作用范围
- 触发场景
- 渐进式加载路由
- 通用编码规则
- 高价值 gotchas

不要把所有持久层、数据库建模、框架细节都塞进主文件。主文件一旦变胖，就会失去作为单一入口的价值。

经验规则：

- 高频入口 skill 优先保持短小
- 只有默认需要知道的规则才放主文件
- 代理不一定会主动去读深层 reference，因此特别容易踩坑的规则可以保留在主文件里

## 4. 用渐进式 `references/` 控制上下文

当一个 skill 同时覆盖通用编码、持久层、数据库设计等不同深度的内容时，优先采用渐进式加载。

推荐结构：

```text
skills/
  java-code-writing/
    SKILL.md
    references/
      database-conventions.md
      mybatis-plus/
        overview.md
        entity-conventions.md
        dao-patterns.md
        paging-batch-update.md
```

推荐做法：

- 主 skill 先只告诉代理“什么时候读哪个 reference”
- `overview.md` 作为子领域入口，再继续细分
- 每个 reference 只承担一个清晰职责

不要默认拆出没有明显收益的文件，例如：

- `java-general.md`
- `spring-framework.md`

只有当内容已经足够独立、而且不拆会让主文件明显变胖时，才值得单独建文件。

## 5. 结构拆分要匹配任务边界

拆分 reference 时，优先按“任务路由”来拆，而不是按你写文档时的思维顺序来拆。

例如 MyBatis-Plus 最合适的拆分不是一个大而全的 `persistence.md`，而是：

- `entity-conventions.md`
- `dao-patterns.md`
- `paging-batch-update.md`

原因：

- DO 设计任务只需要实体规则
- DAO 查询任务只需要 DAO 规则
- 分页、批量、局部更新任务只需要边界与安全规则

拆分后的文件要满足两个标准：

- 文件职责单一
- 代理能凭任务类型快速判断“该读哪一个”

## 6. 主 skill 要保留高价值 gotchas

渐进式加载不等于把所有细节都下沉。以下这类规则应优先保留在主 skill 中：

- 代理很容易默认做错
- 错了以后影响大
- 代理在出错前未必意识到要去读更深的 reference

这次讨论中适合放在主 skill 的 gotchas 包括：

- 不要默认假设表里有 `deleted`
- 局部更新优先于读整行再 `updateById`
- `Page<T>` 不应泄漏到 controller 边界
- 布尔字段避免 `is_` 前缀
- 时间字段优先 `*_time`

## 7. 每个深层 reference 都要带 `Self-Check`

深层 reference 不是只提供规则，还应帮助代理在输出前自检。

例如：

- `entity-conventions.md` 检查 DO 命名、基类选择、是否误加 `deleted`
- `dao-patterns.md` 检查 DAO 是否仍然足够薄
- `paging-batch-update.md` 检查分页边界、批量分片、局部更新安全
- `database-conventions.md` 检查主键、关系字段、时间字段、长文本拆表

这样做的价值是：

- 让深层 reference 不只是知识库
- 让代理在提交前有一个最后校验点

## 8. 用真实任务回放验证 skill，而不是只检查文件结构

skill 文档写完后，不要只验证这些事情：

- 文件存在
- 链接没坏
- frontmatter 合法

还要验证它是否真的按预期路由。

建议至少准备 5 类真实任务：

1. 只写 Service 的任务
2. 只改 `*DO` 的任务
3. 只改 DAO 查询或 default method 的任务
4. 分页、批量、局部更新任务
5. 数据库建模任务

验证目标：

- Service-only 任务不应被 persistence 规则污染
- DO 任务应进入 entity reference
- DAO 任务应进入 dao reference
- 分页批量任务应进入 paging reference
- 建模任务应进入 database reference

如果实际路由不符合预期，就继续收紧 `SKILL.md` 的触发文案。

## 9. Skill 创建流程应当迭代，不要一次性定稿

推荐流程：

1. 先明确目标和边界
2. 先写最小可用版本
3. 用真实任务回放发现误触发和漏触发
4. 修正文案和拆分结构
5. 再补 gotchas、自检、示例

不要一开始就追求“大而全”的技能体系。先确保：

- 触发准确
- 上下文够省
- 深层规则找得到
- 代理真正会用

## 10. 这次沉淀出的核心原则

- `description` 只写触发条件
- 主 `SKILL.md` 保持轻量
- 低频重细节放到 `references/`
- 拆分按任务边界，不按写作者思路
- 主 skill 保留高价值 gotchas
- 每个深层 reference 都应带 `Self-Check`
- 用真实任务回放验证路由效果

这些规则共同服务于同一个目标：

- 让 skill 更容易被正确触发
- 让代理在默认上下文中只加载真正需要的信息
- 让生成结果更稳定、更可维护、更贴近团队约束
