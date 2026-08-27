# 知识分类

## 知识类型（`type`）

每条正式知识必须在 frontmatter 中声明一个 `type`：

| `type` | 含义 | 示例 |
|---|---|---|
| `concept` | 概念性知识："X 是什么" | PLL（锁相环）是什么 |
| `procedure` | 操作步骤："如何做 X" | 如何配置 STM32 Clangd |
| `troubleshooting` | 问题排查记录 | WSL npm 网络错误排查 |
| `decision` | 设计/技术决策及理由 | 为什么 ADC 采用 TIM1 触发 |
| `reference` | 速查型参考资料（手册摘录、速查表等） | 寄存器地址表、命令速查 |
| `paper` | 论文笔记：问题、方法、结论 | 某篇 sensorless 控制论文的笔记 |
| `project` | 项目相关的整体记录 | 某个具体项目的背景与进展 |
| `experiment` | 实验条件和结果 | 某次电机参数辨识实验记录 |

选择原则：优先选最贴近内容本质的一个 `type`；如果一条笔记同时像 `concept` 又像
`procedure`，通常说明应该拆成两条笔记，或者以内容的主要意图为准（"讲清楚是什么"选
`concept`，"讲清楚怎么做"选 `procedure`）。

## `domain` / `subdomain` 与目录的对应关系

`domain` 字段建议使用小写、连字符分隔的写法，并与笔记实际存放的领域目录对应：

| 目录 | 建议 `domain` 取值 |
|---|---|
| `10-Embedded/` | `embedded` |
| `20-Motor-Control/` | `motor-control` |
| `30-Control-Theory/` | `control-theory` |
| `40-MATLAB-Simulink/` | `matlab-simulink` |
| `50-Toolchains/` | `toolchains` |
| `60-System/` | `system` |
| `70-Papers/` | `papers` |
| `80-Projects/` | `projects` |

`subdomain` 是 `domain` 下更细的分类，视具体内容自由命名（例如 `domain: motor-control`
下可以有 `subdomain: sensorless`、`subdomain: foc` 等），不强制枚举——但同一个 `subdomain`
应该在整个 Vault 中保持拼写一致，新建笔记前先搜索一下已有笔记用的是什么写法。
