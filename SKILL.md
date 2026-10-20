---
name: ralph
user-invocable: true
description: 把 PRD 需求文档转换成 Ralph 自主执行系统要用的 prd.json 格式（userStories 数组，每条带可验证验收标准与 passes 标志）。触发词：convert this prd、turn this into ralph format、create prd.json from this、ralph json、转成 ralph 格式、生成 prd.json、需求转故事、拆 ralph 任务。
---

# Ralph PRD 转换器（汉化版）

> 上游：snarktank/ralph 的 Ralph PRD Converter skill（MIT）。本版全文汉化，并把上游
> 生态专属词（Amp / dev-browser / Typecheck）换成通用写法。

把现有 PRD（markdown 或一段文字）转换成 Ralph 自主执行循环吃的 `prd.json`。

## 输出格式

```json
{
  "project": "[项目名]",
  "branchName": "ralph/[feature-name-kebab-case]",
  "description": "[取自 PRD 标题/简介的功能描述]",
  "userStories": [
    {
      "id": "US-001",
      "title": "[故事标题]",
      "description": "作为[角色]，我想要[功能]，以便[价值]",
      "acceptanceCriteria": [
        "标准 1",
        "标准 2",
        "编译/静态检查通过"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

## 第一铁律：故事粒度

**每条故事必须在一轮 Ralph 迭代（一个上下文窗口）内做完。**

Ralph 每轮起新进程、无前史。故事太大，LLM 没做完就断上下文，产出残废代码。

### 合适的粒度：
- 加一个数据库列 + 迁移
- 给既有页面加一个组件
- 给服务端方法加一段新逻辑
- 给列表加一个筛选下拉

### 太大（要拆）：
- "做整个仪表盘" → 拆成：表结构、查询、UI 组件、筛选
- "加登录认证" → 拆成：表结构、中间件、登录页、会话处理
- "重构 API" → 每个端点/每类模式一条故事

**经验法则：两三句话说不清楚怎么改，就是太大。**

## 故事排序：依赖先行

故事按 priority 顺序执行。前面的故事不许依赖后面的。

**正确顺序：**
1. 表结构/数据库变更（迁移）
2. 服务端/后端逻辑
3. 用到后端的 UI 组件
4. 聚合数据的汇总视图

**错误顺序：**
1. UI 组件（依赖还不存在的表结构）
2. 表结构变更

## 验收标准：必须可验证

每条标准都要是 Ralph 能**查**的，不许含糊。

### 好的标准（可验证）：
- "tasks 表加 status 列，默认 'pending'"
- "筛选下拉有选项：全部、进行中、已完成"
- "点删除先弹确认框"
- "编译/静态检查通过"
- "单测通过"

### 坏的标准（含糊）：
- "正常工作"
- "用户用起来方便"
- "体验好"
- "处理好边界情况"

### 每条故事必含的最后一条：
```
"编译/静态检查通过"
```

有可测逻辑的故事再加：
```
"单测通过"
```

改 UI 的故事再加：
```
"用浏览器实测验证"
```

前端故事没经过浏览器实测不算完成。Ralph 会导航到页面、操作 UI、确认改动生效。

## 转换规则

1. **每条用户故事对应一条 JSON 记录**
2. **ID**：顺序编号（US-001、US-002……）
3. **priority**：先按依赖顺序，再按文档出现顺序
4. **所有故事**：`passes: false`，`notes` 留空
5. **branchName**：按功能名 kebab-case，前缀 `ralph/`
6. **必加**：每条故事验收标准末尾加"编译/静态检查通过"

## 大 PRD 拆分

PRD 里的大功能要拆：

**原文：**
> 加用户通知系统

**拆成：**
1. US-001: 建通知表
2. US-002: 建通知发送服务
3. US-003: 页头加通知铃铛
4. US-004: 通知下拉面板
5. US-005: 标记已读
6. US-006: 通知偏好设置页

每条都是能独立完成、独立验证的一个聚焦改动。

## 示例

**输入 PRD：**
```markdown
# 任务状态功能

给任务加状态标记。

## 需求
- 列表上可切换 pending/in-progress/done
- 按状态筛选
- 状态有徽标显示
- 状态落库
```

**输出 prd.json：**
```json
{
  "project": "TaskApp",
  "branchName": "ralph/task-status",
  "description": "任务状态功能 - 带状态标识的任务状态跟踪",
  "userStories": [
    {
      "id": "US-001",
      "title": "tasks 表加 status 字段",
      "description": "作为开发者，我需要持久化任务状态。",
      "acceptanceCriteria": [
        "加 status 列：'pending' | 'in_progress' | 'done'（默认 'pending'）",
        "迁移脚本执行成功",
        "编译/静态检查通过"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "任务卡显示状态徽标",
      "description": "作为用户，我想一眼看到任务状态。",
      "acceptanceCriteria": [
        "每张卡显示彩色状态徽标",
        "颜色：灰=pending、蓝=in_progress、绿=done",
        "编译/静态检查通过",
        "用浏览器实测验证"
      ],
      "priority": 2,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-003",
      "title": "列表行加状态切换",
      "description": "作为用户，我想直接在列表里改任务状态。",
      "acceptanceCriteria": [
        "每行有状态下拉或开关",
        "改状态立即保存",
        "界面无刷新更新",
        "编译/静态检查通过",
        "用浏览器实测验证"
      ],
      "priority": 3,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-004",
      "title": "按状态筛选",
      "description": "作为用户，我想按状态过滤列表。",
      "acceptanceCriteria": [
        "筛选下拉：全部 | Pending | In Progress | Done",
        "筛选状态持久化在 URL 参数",
        "编译/静态检查通过",
        "用浏览器实测验证"
      ],
      "priority": 4,
      "passes": false,
      "notes": ""
    }
  ]
}
```

## 归档上一轮

**写新 prd.json 前，先看有没有别的功能的旧文件：**

1. 读现有 `prd.json`（如果有）
2. 看 `branchName` 是否和新功能的分支名不同
3. 若不同且 `progress.txt` 有头部以外的内容：
   - 建归档目录：`archive/YYYY-MM-DD-feature-name/`
   - 把当前 `prd.json` 和 `progress.txt` 拷进去
   - `progress.txt` 重置为全新头部

**ralph.sh 跑的时候会自动做这件事**；若你在两轮之间手动更新 prd.json，请先归档。

## 保存前检查

- [ ] 上一轮已归档（prd.json 存在且 branchName 不同，先归档）
- [ ] 每条故事小到一轮能做完
- [ ] 故事按依赖排序（先表结构后后端后 UI）
- [ ] 每条故事都有"编译/静态检查通过"
- [ ] UI 故事都有"用浏览器实测验证"
- [ ] 验收标准可验证（不含糊）
- [ ] 没有故事依赖排在它后面的故事
