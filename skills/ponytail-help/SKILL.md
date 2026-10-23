---
name: ponytail-help
description: >
  ponytail 的速查卡：强度档位、各个 Skill 分别做什么、怎么开关、怎么改默认档位。一次性展示，不改任何东西。
  当用户说"ponytail 怎么用""有什么命令""速查表"时使用，也可直接调用 ponytail:ponytail-help。
---

# Ponytail Help

被调用时展示下面这张速查卡。**一次性任务：不要**切换模式、写状态文件或持久化任何东西。

## 强度档位

| 档位 | 触发 | 行为变化 |
|------|------|---------|
| **lite** | `ponytail:ponytail lite` | 按要求实现，用一行点出更小的方案让用户选。 |
| **full** | `ponytail:ponytail` | 最小且完整的改动、该有检查的地方有检查、回复里说明跳过了什么和有什么风险。默认档位。 |
| **ultra** | `ponytail:ponytail ultra` | 额外质疑需求本身，动手前先反驳。先想删除，再想新增。 |

档位在**本次会话的后续任务**中保持；宿主没有 Hook，新会话需要重新指定。`ponytail:ponytail` 不带参数时报告当前档位；当前是 `off` 时则按默认档位开启。

> 如果宿主的 Skill 调用不支持带参数（投稿指南未记载这一点），用户直接说「ponytail 切到 lite 档」「用 ultra 模式」即可，效果相同。

## Skill 列表

| Skill | 触发方式 | 做什么 |
|-------|---------|--------|
| **ponytail** | `ponytail:ponytail` | 极简模式本体：最少的新代码、每条回复都讲清跳过了什么和风险在哪。 |
| **ponytail-review** | `ponytail:ponytail-review` | 对改动做质量评审：bug、安全、真实负载、缺失的测试、速度、能删的代码。每条发现都讲清会出什么问题和怎么修。 |
| **ponytail-audit** | `ponytail:ponytail-audit` | 同样的质量评审，范围是整个仓库，按重要性排序。 |
| **ponytail-debt** | `ponytail:ponytail-debt` | 把 `shortcut:` 注释汇成可追踪的债台账。 |
| **ponytail-gain** | `ponytail:ponytail-gain` | 实测收益计分板：更少代码、更低成本、更快。 |
| **ponytail-help** | `ponytail:ponytail-help` | 就是这张卡。 |

在 MiniMax Code 里，直接在对话里说「用 ponytail-review 审一下当前改动」也可以，Agent 会自己选中对应 Skill。

## 关闭与恢复

说「关掉 ponytail」或「恢复正常模式」，或用 `ponytail:ponytail off`。任何时候用 `ponytail:ponytail` 恢复。

## 改默认档位

默认是 `full`。想改：

**环境变量**（优先级最高）：

```bash
export PONYTAIL_DEFAULT_MODE=ultra
```

**配置文件**（`~/.config/ponytail/config.json`，Windows 是 `%APPDATA%\ponytail\config.json`）：

```json
{ "defaultMode": "lite" }
```

取值 `off | lite | full | ultra`。优先级：环境变量 > 配置文件 > `full`。

> **这个宿主不会自动读它们。** MiniMax Plugin 不受理 Hook，MiniMax Code 也不会在会话启动时去读这两个文件——它们只在你手动调用本 Skill、Agent 读到这段说明时才会被看到。所以「设成 off 就不会自动进入本模式」在这里**不成立**：真正决定是否进入本模式的，是 Agent 有没有选中这个 Skill。
>
> 想要硬性常驻，唯一可靠的办法是把本插件包里的 `AGENTS.md` 复制到你的项目根目录——MiniMax Code 会把项目根的 `AGENTS.md` 当作项目指令读取（若你的版本没有自动加载，手动引用该文件即可）。

## 上游与致谢

上游项目：https://github.com/DietrichGebert/ponytail （MIT 协议，作者 Dietrich Gebert）。本插件是面向 MiniMax Code 的移植版，保留了原始规则集与判定标准，去掉了仅适用于其他宿主的部分（生命周期 Hook、状态栏 Band 功能）。
