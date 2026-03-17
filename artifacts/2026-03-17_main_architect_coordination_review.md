# Main Agent 与 Architect Agent 协调调用经验总结

**日期**: 2026-03-17
**作者**: main agent (女娲 🧚)

---

## 📋 执行的操作

### 1. snake-game 项目创建
- ✅ 创建项目目录结构
- ✅ 创建 project.json
- ✅ 创建飞书群组
- ✅ 配置 architect agent binding
- ✅ 重启 Gateway

### 2. 发现映射关系问题
- ❌ 项目群组未关联到 project_name
- ❌ architect agent 无法识别项目

### 3. 跨群组调用尝试
- ❌ sessions_spawn 被拒绝（权限限制）
- ❌ 跨群组自动触发不现实

### 4. PROMPT.md 修正
- ✅ 修正步骤 9（引导用户切换）
- ✅ 添加步骤 6.1（记录项目群组映射）
- ✅ 更新执行顺序

### 5. 项目更新
- ✅ etf-quant：标准格式
- ✅ openclaw-notes：标准格式
- ✅ 删除 rete 目录

---

## ❌ 踩过的坑

### 坑 1：跨群组自动调用 architect 失败

**现象**:
```json
{
  "error": "agentId is not allowed for sessions_spawn (allowed: none)"
}
```

**根本原因**:
- main agent 不允许通过 sessions_spawn 调用其他 agent
- architect agent 只在项目群组有消息时才启动 session
- 没有消息时，没有 session 可调用

**影响**:
- 无法实现"自动触发 architect"的需求
- 需要用户手动切换到项目群组

**教训**:
- ❌ 不要设计复杂的跨群组调用机制
- ✅ 让用户主动切换到项目群组
- ✅ architect 在项目群组中自动响应

---

### 坑 2：项目群组映射缺失

**现象**:
- 创建了项目群组（`oc_ce352bbc2f7a45e8111e62687a947313`）
- project_mapping.json 只记录了协调群组
- architect agent 在项目群组中查不到 project_name

**根本原因**:
- PROMPT.md 步骤 5.2 只记录"当前群组"（协调群组）
- 步骤 6 创建项目群组后，没有更新 project_mapping.json
- 流程遗漏了项目群组映射

**影响**:
- architect agent 无法识别项目
- 用户消息无响应

**教训**:
- ❌ 流程设计不完整
- ✅ 创建项目群组后必须更新映射
- ✅ 需要记录两个群组的映射关系

**修复**:
- 添加步骤 6.1：记录项目群组映射
- 格式：
```json
{
  "协调群组chat_id": "project_name",
  "项目群组chat_id": "project_name"
}
```

---

### 坑 3：项目目录结构不统一

**现象**:
- etf-quant：旧格式，无标准目录结构
- rete：旧格式，无 project.json
- snake-game：新格式，标准目录结构

**根本原因**:
- 旧项目没有按照新的 PROMPT.md 创建
- 缺少标准化的 project.json
- 目录结构不一致

**影响**:
- architect agent 无法识别项目
- 需要手动更新项目信息

**教训**:
- ❌ 项目迁移容易遗漏
- ✅ 所有项目应该遵循统一格式
- ✅ 需要项目规范化脚本

**修复**:
- 更新 etf-quant 为标准格式
- 删除 rete（不再需要）
- 创建标准 project.json

---

## 🎯 关键发现

### 1. agent 工作机制

**main agent 工作流程**:
```
1. 接收消息（协调群组）
2. 判断是运维还是用户需求
3. 如果用户需求 → 创建项目
4. 创建项目群组
5. 引导用户切换到项目群组
```

**architect agent 工作流程**:
```
1. 等待项目群组消息
2. 收到消息 → 启动 session
3. 查询 project_mapping.json 获取 project_name
4. 使用对应的 workspace
5. 执行技术工作
```

### 2. 群组映射关系

**两个群组的职责**:

| 群组类型 | 用途 | 绑定的 agent |
|---------|------|-------------|
| 协调群组 | 项目管理和讨论 | main agent |
| 项目群组 | 技术架构和开发 | architect agent |

**project_mapping.json 设计**:
```json
{
  "note": "只有项目群组在映射中",
  "projects": {
    "项目群组chat_id": "project_name"
  }
}
```

**为什么协调群组不在映射中？**
- main agent 在协调群组中工作
- 不需要查询 project_name（因为不在项目群组）
- 协调群组用于创建新项目，不是已有项目

### 3. 跨群组调用不现实的原因

**architect agent 的启动时机**:
- 通过 binding 绑定到项目群组
- **只在项目群组有消息时才启动 session**
- 没有消息时，architect agent 的 session 不存在

**sessions_spawn 的限制**:
- main agent 不允许调用其他 agent
- 即使可以调用，目标 session 也不存在

**跨群组消息发送的问题**:
- 可以通过 message 工具发送
- 但 architect 的响应只在项目群组中
- 用户看不到完整的对话流程

---

## 📋 改进需求

### 需求 1：项目规范化脚本

**目标**:
统一所有项目为标准格式

**功能**:
```bash
./scripts/normalize-project.sh <project_name>
```

**操作**:
1. 检查项目目录结构
2. 创建标准目录（memory/artifacts/logs）
3. 创建/更新 project.json
4. 更新 project_mapping.json
5. 验证完整性

**输入参数**:
- project_name: 要规范化的项目名称

---

### 需求 2：映射关系验证工具

**目标**:
确保 project_mapping.json 与实际绑定一致

**功能**:
```bash
./scripts/verify-mapping.sh
```

**检查项**:
1. project_mapping.json 中的项目群组是否都在 bindings 中
2. bindings 中的项目群组是否都有对应的 project
3. 目录结构是否完整
4. project.json 是否存在

**输出报告**:
```json
{
  "total_projects": 3,
  "valid_projects": 2,
  "issues": [
    {
      "project": "unknown",
      "chat_id": "oc_be7c4d135d1ff2fb5ccd92ad287175c0",
      "issue": "找不到对应的项目目录"
    }
  ]
}
```

---

### 需求 3：项目清理工具

**目标**:
删除不需要的项目和清理残留

**功能**:
```bash
./scripts/cleanup-project.sh <project_name>
```

**操作**:
1. 删除项目目录
2. 从 project_mapping.json 中移除映射
3. 从 openclaw.json 中移除 binding
4. 解散飞书群组
5. 重启 Gateway

**安全措施**:
- 两次确认
- 提示将要删除的内容
- 支持撤销（30秒内可取消）

---

### 需求 4：项目信息查看工具

**目标**:
快速查看项目状态和配置

**功能**:
```bash
./scripts/list-projects.sh
```

**输出格式**:
```
=== 项目列表 ===

1. snake-game
   状态: running
   阶段: architect
   目录: /root/workspace/snake-game
   协调群组: oc_bdfc684bcc22edc585e55ff7dcbc42ce
   项目群组: oc_ce352bbc2f7a45e8111e62687a947313

2. etf-quant
   状态: running
   阶段: architect
   目录: /root/workspace/etf-quant
   项目群组: oc_1b61e657caadedc8b1e00fe303c95f22
```

---

## 🎓 最佳实践

### 1. 项目创建流程

```
1. main agent（协调群组）→ 收集需求
2. main agent（协调群组）→ 创建项目
3. main agent（协调群组）→ 创建项目群组
4. main agent（协调群组）→ 配置 binding
5. main agent（协调群组）→ 记录两个群组的映射
6. main agent（协调群组）→ 重启 Gateway
7. main agent（协调群组）→ 引导用户切换
8. 用户（手动）→ 切换到项目群组
9. architect（项目群组）→ 自动响应
```

### 2. 映射关系管理

**何时更新 project_mapping.json**:
- ✅ 创建新项目时（两个群组都要映射）
- ❌ 不需要映射协调群组
- ✅ 删除项目时（移除项目群组映射）

**记录格式**:
```json
{
  "projects": {
    "项目群组chat_id": "project_name"
  }
}
```

### 3. PROMPT.md 设计原则

**避免**:
- ❌ 设计复杂的跨群组调用
- ❌ 假设 agent session 始终存在
- ❌ 流程不完整（遗漏关键步骤）

**遵守**:
- ✅ 每个步骤都要明确
- ✅ 关键操作必须有验证
- ✅ 考虑失败场景
- ✅ 提供回滚机制

---

## 🔧 技术改进建议

### 1. main agent PROMPT.md 优化

**当前问题**:
- 项目创建流程不够健壮
- 缺少错误处理和验证

**改进点**:
1. 添加项目名称重复检测
2. 添加项目名称规范化（slugify）
3. 添加项目目录冲突检测
4. 添加回滚机制

### 2. architect agent 能力增强

**当前限制**:
- 只能在项目群组中响应
- 无法跨项目共享知识

**改进方向**:
1. 支持项目间知识共享
2. 支持项目模板系统
3. 支持快速启动新项目

### 3. 工具化

**需要开发的脚本**:
1. `normalize-project.sh` - 项目规范化
2. `verify-mapping.sh` - 映射验证
3. `cleanup-project.sh` - 项目清理
4. `list-projects.sh` - 项目列表
5. `migrate-project.sh` - 项目迁移

---

## 📊 统计数据

### 今日操作统计

| 操作类型 | 次数 | 成功 | 失败 |
|---------|------|------|------|
| 项目创建 | 1 | 1 | 0 |
| 群组创建 | 1 | 1 | 0 |
| Gateway 重启 | 6 | 6 | 0 |
| PROMPT.md 更新 | 2 | 2 | 0 |
| 项目更新 | 2 | 2 | 0 |
| 目录删除 | 1 | 1 | 0 |

### 踩坑统计

| 坑类型 | 发现次数 | 已修复 | 未修复 |
|---------|---------|--------|--------|
| 跨群组调用失败 | 1 | 1 | 0 |
| 项目群组映射缺失 | 1 | 1 | 0 |
| 项目目录结构不统一 | 1 | 1 | 0 |
| 流程遗漏 | 1 | 1 | 0 |

---

## 🎯 下一步行动

1. **immediate** (本周期)
   - ✅ 完成经验总结
   - ✅ 形成需求文档
   - ⏳ 将文档输出到 GitHub Pages

2. **short-term** (下周)
   - 开发项目规范化脚本
   - 开发映射验证工具
   - 完善 PROMPT.md 错误处理

3. **long-term** (本月)
   - 实现项目模板系统
   - 实现 agent 能力增强
   - 建立完整的工具链

---

**总结者**: main agent (女娲 🧚)
**日期**: 2026-03-17
**文档版本**: v1.0
