# scenario-test

基于 Gherkin 的测试用例资产化与并行执行系统，支持 AI 驱动的自然语言测试。

## 包含的资产

| 类型 | 名称 | 说明 |
|------|------|------|
| **Skill** | `scenario-test` | 管理 Gherkin `.feature` 测试用例（增删改查、去重校验、编写规范） |
| **Command** | `exec-scenarios` | 执行 Gherkin 测试用例，支持并行执行、智能重试、结果汇总 |
| **Sub-Agent** | `scenario-case-runner` | 测试用例执行子代理，接收指令包并逐步调用工具执行 |

## 安装

```bash
npx agent-get \ 
  --skill git@github.com:pea3nut/scenario-test.git#skills/scenario-test \
  --command git@github.com:pea3nut/scenario-test.git#commands/exec-scenarios.md \
  --sub-agent git@github.com:pea3nut/scenario-test.git#agents/scenario-case-runner.md
```

## 支持的宿主

需要宿主支持 skill、command、sub-agent 三种资产类型。推荐使用 **Cursor** 或 **Claude Code**。

## 使用说明

### 1. 管理测试用例（Skill: scenario-test）

安装后，AI Agent 会自动识别 `scenario-test` skill，支持以下操作：

- **添加用例**：提供 Gherkin 文本，AI 自动写入 `.feature` 文件并去重
- **更新用例**：按 Scenario 名称定位并更新
- **列出用例**：支持按目录、tags、关键字过滤
- **读取用例**：查看单个用例的完整内容
- **编写规范**：询问 Gherkin 写法，获取语法说明和示例

### 2. 执行测试（Command: /exec-scenarios）

```
/exec-scenarios [cases=<路径或片段>] [config=<配置文件路径>]
```

**首次执行**会自动生成 `scenario-run-config.md` 配置文件，需要填写：
- `executor.command`：执行器（`playwright-cli`、`bash-curl` 或自定义 CLI）
- `executor.config`：用自然语言描述执行器配置（目标地址、认证方式等）

**执行流程**：
1. 嗅探测试目录和配置
2. 扫描 `.feature` 文件，发现所有 Scenario
3. 批量并行执行（通过 `scenario-case-runner` 子代理）
4. 智能重试失败用例
5. 生成 `result.json` + `summary.md` 结果报告

### 3. 支持的执行器

| 执行器 | 用途 | 并发隔离方式 |
|--------|------|-------------|
| `playwright-cli` | 浏览器 UI 自动化（默认） | 独立 session（`-s=<caseId>`） |
| `bash-curl` | HTTP 接口测试 | 天然无状态 |
| 自定义 CLI | vitest、jest 等测试框架 | 临时目录隔离 |

## 目录结构

```
scenario-test/
├── manifest.json
├── README.md
├── skills/
│   └── scenario-test/
│       ├── SKILL.md                   # 用例管理 skill 定义
│       ├── mixed-case-template.md     # MixedCase 指令包格式模板
│       └── run-config-template.md     # 运行配置模板
├── commands/
│   └── exec-scenarios.md              # 执行命令定义
└── agents/
    └── scenario-case-runner.md        # 子代理定义
```
