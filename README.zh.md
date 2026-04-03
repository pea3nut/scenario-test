# scenario-test

[English](README.md) | [日本語](README.ja.md)

使用**极其泛化的自然语言**来编写和执行测试用例，0 代码

```gherkin
Scenario: 新用户完成注册
    Given 用户在注册页面
    When 填写所有必填信息并提交注册表单
    Then 注册成功并跳转到用户中心，且信息与填写一致
```

## 安装

```bash
npx -y agent-add \
  --skill https://github.com/pea3nut/scenario-test.git#skills/scenario-test \
  --command https://github.com/pea3nut/scenario-test.git#commands/scenario-exec.md \
  --sub-agent https://github.com/pea3nut/scenario-test.git#agents/scenario-case-runner.md
```

scenario-test 依赖于 skill、command、sub-agent 的支持。请确保你使用的 AI 编程工具支持

## 生成用例

安装后 AI 会自动识别 scenario-test 的 skill，用自然语言描述你要测什么即可

场景一：新功能开发过程中

```md
/speckit.specify 开发 xxx 功能，测试部分使用 scenario-test 
```

场景二：对已有功能产出 case

```md
帮我针对所有的登陆相关接口生成 scenario-test 的用例，功能覆盖完全
```

## 运行配置

首次生成用例时，AI 会自动创建 `scenario-run-config.md` 并根据项目上下文推断配置。你可以在运行前检查和修改。

| 配置项        | 默认值 | 说明                       |
| ------------- | ------ | -------------------------- |
| `concurrency` | 4      | 最大并发子代理的数量       |
| `maxFailures` | 10     | 失败数达到此值后提前终止   |
| `maxRetries`  | 1      | 智能重试次数（0 = 不重试） |

## 运行用例

其中最关键的是**执行器配置**——scenario-test 需要知道如何执行一个测试用例，比如将自然语言用例转换为基于 curl 的接口测试、或基于 Playwright CLI 的 E2E 测试

一般来说，scenario-test 会自动根据项目背景判断，自动填充执行器。

其余配置项均有默认值，按需调整即可：

```md
/scenario-exec {测试用例文件或文件夹} {可选，运行配置文件。默认会自动选择} 
```

执行时采用**双模型架构**，来最大化质量、成本和执行速度：

- 主模型（即你正在对话的模型）负责理解用例、翻译为执行器指令、分析失败原因并决定是否重试
- 子代理模型（即主模型拉起的多个子代理任务）负责执行实际的测试步骤（打开浏览器、发请求、点按钮……）

所以你需要用一个**聪明的模型**（如 Claude Sonnet）来运行这条命令，子代理会自动使用廉价模型

## 适用场景

Scenario Test 基于自然语言描述用例，因此天然适合**功能级别的测试**，可以支持很泛化的描述，如：

- 补充所有必填字段
- 通过 Get 接口返回的字段和我们提交相同
- 在列表页面搜索到我们刚刚创建的记录

一旦你的测试接近于实现层面，如某个函数的实现，那么你能用的自然语言泛化能力则越少，Scenario Test 带来的优势也就越少

