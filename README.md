# ralph — PRD 转 prd.json 转换器

把 PRD 需求文档转换成 Ralph 自主执行系统吃的 `prd.json`（snarktank/ralph 上游汉化版）：每条故事一个上下文窗口能做完、验收标准机器可查、依赖先行排序。

## 相关技能

- [agent-config-sync-check](https://github.com/huzhw/agent-config-sync-check)：八端同步守卫：链接/硬链接/README 同步检查与修复
- [git-commit](https://github.com/huzhw/git-commit-skill)：Git 提交规范
- [daily-record-gitlab-md](https://github.com/huzhw/daily-record-gitlab-md-skill)：日报记录（第一步）
- [daily-merge-gitlab-excel](https://github.com/huzhw/daily-merge-gitlab-excel-skill)：日报合并（手动兜底，已由日报管家接管，不推荐）
- [code-check](https://github.com/huzhw/code-check-skill)：增量代码隐患检查
- [deepseek-harness-settings-curator](https://github.com/huzhw/deepseek-harness-settings-curator)：DeepSeek Harness 模型配置梳理
- [deepseek-harness-plugin-doctor](https://github.com/huzhw/deepseek-harness-plugin-doctor)：DeepSeek Harness 插件与升级体检医生
- [reread-rules](https://github.com/huzhw/reread-rules-skill)：重载 CLAUDE.md / AGENTS.md 规则
- [dev-env-upgrade](https://github.com/huzhw/dev-env-upgrade)：开发环境升级医生：版本体检/最新版查询/安全升级
- [prd](https://github.com/huzhw/prd-skill)：PRD 生成器（一句话需求→结构化需求文档）
- [coding-rules](https://github.com/huzhw/coding-rules)：编码规则库（独立仓库，非 skill）
- [daily-report-panel](https://github.com/huzhw/daily-report-panel)：日报管家（关联仓库，非 skill，自动合并/导出/发件）

## 解决了什么问题

Ralph 每轮起全新 agent 进程（无前史、抗上下文腐化），吃的不是 markdown 而是结构化 `prd.json`。手写 JSON 容易犯三类错：故事太大（一个上下文窗口做不完，产出残废代码）、验收标准含糊（agent 说"做完了"无法机械判定）、依赖倒挂（UI 故事排在表结构前面）。这个技能把 PRD 机械转换为合规格的 prd.json，并强制三条铁律：一轮一故事、标准可验证、依赖先行。

## 使用

对 AI 说触发词即可：

- "转成 ralph 格式"、"生成 prd.json"、"需求转故事"、"拆 ralph 任务"
- "convert this prd"、"turn this into ralph format"、"ralph json"

AI 按模板转换、拆分大故事、按依赖排序，产出可直接被 `ralph.sh` 消费的 prd.json。

## 文件结构

```
ralph/
├── SKILL.md           # 技能本体（frontmatter name/description + 转换规则与示例）
├── README.md          # 本文件
├── JUNCTION说明.md    # 五端 Junction 指向关系（agent-config-sync-check 自动维护）
└── .gitignore
```

## 安装

```bash
git clone https://github.com/huzhw/ralph-skill.git ~/.claude/skills/ralph
```

重启 Claude Code 生效。

## 许可

MIT（上游 snarktank/ralph 同 license）
