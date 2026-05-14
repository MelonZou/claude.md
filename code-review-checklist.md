# 代码审查检查清单

> 每次 Codex 完成后，审查前必须读取此文件，逐项执行。持续积累，随时更新。

---

## 通用检查

- [ ] `git diff --stat HEAD` 确认所有改动文件范围，不遗漏
- [ ] **改动涉及页面/组件时，必须往上追溯入口**：确认实际调用方（路由跳转、父组件引用）与改动文件一致，防止改了不会被触发的文件。具体做法：grep 改动文件名在整个项目中的引用，并确认入口传参（如 `currentType`）与改动中的条件判断匹配
- [ ] 新增方法：null 参数处理、边界条件是否覆盖
- [ ] 异常类型是否符合项目规范（Spring Boot 项目用 `RcBizException`，禁止抛 `IllegalArgumentException` 等原生异常）
- [ ] SQL / Mapper：参数为 null 时语义是否正确（PostgreSQL `col = NULL` 永远 false，需 `<if test>` 处理）
- [ ] 跨表写操作是否加 `@Transactional`

---

## Spring Bean 专项（强制，无例外）

**触发条件（任一成立即必须执行）**：
- 新增 @Service / @Component / @Repository
- **已有 Service 新增 / 修改了任何注入字段**（不论是构造注入 `private final` 还是 `@Autowired` 字段注入）
- 已有 Service 改了 setter 注入或 @Resource 注入

> ⚠️ "已有 Service 加新依赖" 是循环依赖最高发场景，比新建 Bean 更隐蔽，**必须重点查**。本规则曾因触发条件只覆盖"新增 Bean"而漏检，2026-05-12 扩大覆盖到所有依赖变更。

**必查项**：

- [ ] 列出本次改动涉及的每个 Bean 的全部注入依赖（构造 `private final` + `@Autowired` 字段 + setter / @Resource，全部列）
- [ ] **显式画出 A→B→A 的依赖图**，存在环 → 必须打回。Spring Boot 2.6+ 默认 `allow-circular-references=false`，环存在即启动失败
- [ ] 检查混合注入（同一 Bean 同时存在构造注入 + 字段注入）：构造环可能直接启动失败；字段环靠延迟初始化可能侥幸通过，但仍属违规，一并打回
- [ ] 解环优先方案：把 Service→Service 改成 Service→Mapper（Mapper 不参与 Service 互引环）

---

## 前端 Vue 专项（有新增/修改 Vue 组件时）

- [ ] 同类型多个 Tab / 表格中，`fixed="right"` 等样式属性是否一致
- [ ] API 调用失败是否有 `ElMessage.error` 提示，不能静默失败
- [ ] 关键操作（删除/移除）是否有二次确认弹窗

---

## MyBatis XML 专项

- [ ] `<foreach>` 集合为空时是否会生成非法 SQL（如 `IN ()`）
- [ ] 结果映射 resultMap 是否覆盖所有新增字段
- [ ] 新增查询方法是否在 Mapper 接口中声明

---

## 数据库变更专项（有 DDL 改动时必查）

- [ ] 涉及数据库结构变更（新增字段、新建表、改列类型等）时，必须用 Python 连接目标数据库确认变更已实际执行，不能仅依赖 DDL 文件或编译通过
