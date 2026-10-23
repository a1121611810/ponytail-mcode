# Ponytail 极简工程师（MiniMax Code 插件）

> "他什么也不说，写一行代码，搞定了。"

让 MiniMax Code 里的 Agent 像那个**最懒的资深工程师**一样干活：他看一眼你写的五十行，什么也不说，换成一行。

上游项目：[DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) · MIT 协议 · 作者 Dietrich Gebert。本插件是面向 MiniMax Code 的移植版。

## 它解决什么问题

AI Agent 写代码时有个通病：**过度设计**。你要一个日期选择器，它给你装一个日期选择器库、写一个包装组件、加一份样式表，然后开始讨论时区。

Ponytail 的做法是：写代码之前先在"必要性阶梯"上停在够用的那一级。

```
1. 这东西需要存在吗？        → 不需要就跳过（YAGNI）
2. 代码库里已经有了吗？      → 有就复用，别重写
3. 标准库能做吗？            → 用它
4. 平台原生能做吗？          → 用原生（如 <input type="date">）
5. 已装的依赖能做吗？        → 用它
6. 能写成一行吗？            → 那就一行
7. 到这里才：写能跑通的最小实现
```

**但它砍的从来不是正确性。** 信任边界上的校验、防数据丢失的错误处理、安全、可访问性、用户明确要求的一切，永远不砍。代码变小是因为它必要，不是因为在玩代码高尔夫。

## 怎么用

安装后在对话里直接说：

| 你可以说 | 会触发 |
|---------|--------|
| "用 ponytail-review 审一下当前改动" | `ponytail:ponytail-review` |
| "审计一下这个仓库" | `ponytail:ponytail-audit` |
| "我推迟了哪些 shortcut？列个台账" | `ponytail:ponytail-debt` |
| "ponytail 能省多少？" | `ponytail:ponytail-gain` |
| "ponytail 怎么用？" | `ponytail:ponytail-help` |
| "关掉 ponytail" / "恢复正常模式" | 关闭本模式 |

编码任务本身（写代码、修 bug、重构、评审、挑依赖）会自动落到 `ponytail` 这个 Skill 上。

### 强度档位

| 档位 | 行为 |
|------|------|
| `lite` | 按要求实现，用一行点出更小的方案让用户选。 |
| `full` | 默认档位。最小且完整的改动，该有检查的地方有检查。 |
| `ultra` | 额外质疑需求本身，动手前先反驳。 |

切换：直接说「ponytail 切到 lite 档」「用 ultra 模式」「关掉 ponytail」即可。

> MiniMax Code 的 Skill 调用不接收参数，`ponytail:ponytail lite` 这类带参写法不保证生效——用上面那句自然语言说就行，Agent 会自己选中 `ponytail` Skill 并切到对应档位。

> **关于 `PONYTAIL_DEFAULT_MODE` 和 `~/.config/ponytail/config.json`**：这两个是上游给生命周期 Hook 读的配置。MiniMax Plugin 不受理 Hook，MiniMax Code 也不会在会话启动时自动去读它们——它们只有在你手动调用本 Skill、Agent 读到时才会被看到。所以「设成 off 就不会自动进入本模式」在这里不成立：真正决定是否进入本模式的，是 Agent 有没有选中这个 Skill。

### 想让它每个会话都生效

本插件包内附有上游 `AGENTS.md` 的**正文**原样副本（仅替换了标题并加了移植说明）。把它复制到你的项目根目录，MiniMax Code 会把项目根的 `AGENTS.md` 当作项目指令读取（若你的版本没有自动加载，手动引用该文件即可）：

```bash
cp AGENTS.md /path/to/your/project/
```

## 六个 Skill

- **`ponytail`** — 极简模式本体。写代码前先列清楚这次改动必须触及哪些位置，然后走必要性阶梯，最后在回复结尾说明跳过了什么、有什么风险。
- **`ponytail-review`** — 像"半夜被电话叫醒的资深开发者"那样评审改动。读改动触及的代码而不只是 diff，查 bug、安全、真实负载、缺失的测试、慢的路径、能删的代码。每条发现四段式：**是什么 / 问题 / 怎么修 / 不修会怎样**。
- **`ponytail-audit`** — 同样的质量评审，范围是整个仓库，最多 20 条，按重要性排序。
- **`ponytail-debt`** — 把 `shortcut:` 注释汇成技术债台账（标出没有触发条件、正在悄悄烂掉的那些）。
- **`ponytail-gain`** — 展示上游基准测试的实测收益计分板。
- **`ponytail-help`** — 速查卡：档位、各 Skill 的作用、开关方式。

## 实测收益（上游数据）

上游在 headless Claude Code（Opus 5.5）上跑 39 个任务 × 5 次的结果，相对于同一个不带该 Skill 的 Agent：

| 指标 | 结果 |
|------|------|
| 代码行数 | ▼ 53% |
| 输出 token | ▼ 45% |
| 成本 | ▼ 26% |
| 耗时 | ▼ 41% |
| 该有测试处的测试率 | 98%（无 Skill 时 68%） |

> 这是**上游在 Claude Code 上的基准平均值**，不是 MiniMax Code 上的实测结果，也不是你当前仓库的数字。想知道当前仓库里真实的、能数出来的数字，用 `ponytail:ponytail-debt`。

## 与上游的差异

| | 上游 ponytail | 本 MiniMax 插件 |
|---|---|---|
| 能力形态 | Skill + 生命周期 Hook + 多个宿主适配器 | **仅 Skill**（MiniMax 投稿表单不受理 Hook） |
| 常驻激活 | Hook 每轮注入规则集 | Skill description 语义匹配 + 可选 `AGENTS.md` |
| 状态栏 Band | 支持（Claude Code 专用） | 已移除 |
| 正文语言 | 英文 | 中文（代码标记与命令保持英文） |

规则集、判定标准、评审格式全部保持一致。

## 包结构

```
ponytail-mcode/
  .minimax-plugin/plugin.json     # MiniMax Plugin 入口（manifest）
  icon.png                        # 512×512 方形图标
  AGENTS.md                       # 上游规则集正文副本，可复制到项目根目录常驻
  skills/<skill-name>/SKILL.md    # 六个 Skill
```

投稿时包根目录必须直接包含 `.minimax-plugin/plugin.json`；manifest 里 `skills` 列出的路径要与实际文件一一对应，未使用的能力保留空数组。

## 协议与致谢

上游 Ponytail 由 Dietrich Gebert 编写，MIT 协议。本移植版保留原始版权声明与协议原文（见 `LICENSE` 与 `NOTICE`）。
