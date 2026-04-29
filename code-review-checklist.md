# 代码审查检查清单

> 每次 Codex 完成后，审查前必须读取此文件，逐项执行。持续积累，随时更新。

---

## 通用检查

- [ ] `git diff --stat HEAD` 确认所有改动文件范围，不遗漏
- [ ] 新增方法：null 参数处理、边界条件是否覆盖
- [ ] 异常类型是否符合项目规范（Spring Boot 项目用 `RcBizException`，禁止抛 `IllegalArgumentException` 等原生异常）
- [ ] SQL / Mapper：参数为 null 时语义是否正确（PostgreSQL `col = NULL` 永远 false，需 `<if test>` 处理）
- [ ] 跨表写操作是否加 `@Transactional`

---

## Spring Bean 专项（有新增 @Service / @Component 时必查）

- [ ] 列出所有新 bean 的构造注入依赖关系（逐个文件检查 `private final` 字段）
- [ ] **显式检查循环依赖**：画出 A→B→A 的依赖图，存在环则必须打回。这是 Spring 启动失败的高频原因，不能等运行时才发现

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
