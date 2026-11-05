# arch-diagram-skill

服务架构图 skill：**自然语言 ↔ Mermaid ↔ 终端风交互 HTML**。

面向任意 agent（Claude Code / Cursor / ZCode / …）安装即用，零依赖、单文件模板。

## 它能做什么

| 方法 | 输入 | 输出 |
|---|---|---|
| nl2mmd | 自然语言描述的架构 | 约定方言的 Mermaid flowchart |
| mmd2html | 该方言的 Mermaid | 单文件交互 HTML |

生成的 HTML 是终端风格服务架构图，自带：

- **三层结构**：服务器 → 服务 → 子服务
- **收合上浮**：调度边锚定最小颗粒；收合节点时边自动上浮到父节点，全部收合即服务器间连线（三档粒度预设 servers / services / leaves）
- **自由画布**：框顶把手拖拽（按住 Alt 全显）、右缘调宽、⟲ 一键复位
- **边面板**：edge log 随粒度刷新、inspector 展开最小颗粒明细（含调度时刻）、session log
- **本地记忆**：布局与收合状态存 localStorage，刷新还原
- **内置编辑**：Alt+单击节点改名称/备注/框色、增删节点；Alt+单击连线改类型/时刻；Alt 显示右上 ⇱ export，弹窗查看/复制最新状态的 Mermaid（高亮）
- 零外部依赖：无 CDN、无字体请求、无构建

## 安装（给 agent 说一句话）

```
Install arch-diagram skill for me by reading
https://github.com/panfu/arch-diagram-skill/blob/main/llms.txt
```

或手动复制两个文件到你的 agent skills 目录（保持相对路径）：

- `SKILL.md` → `<skills-dir>/arch-diagram/SKILL.md`
- `assets/template.html` → `<skills-dir>/arch-diagram/assets/template.html`

常见 `<skills-dir>`：`~/.agents/skills/`（通用/ZCode）、`~/.claude/skills/`（Claude Code）、`~/.cursor/skills/`（Cursor）。

## 使用

```
# 方法一：自然语言 → mermaid
用 arch-diagram 把这段架构转成 mermaid：
A 服务器跑 nginx(反代+静态) 和 API 网关；网关调用 B 服务器上的订单服务；
订单服务下单时写 B 的 PostgreSQL 主库，主库流复制到只读副本；
运维机 cron 每日 02:00 对账、08:00 推报表，BI 只读副本。

# 方法二：mermaid → html
用 arch-diagram 把这个 mermaid 生成交互 html：<粘贴 flowchart>
```

方言约定（线型即类型）：

| 类型 | 线型 | 语义 |
|---|---|---|
| sync | `==>` | 数据同步 / 写回 |
| call | `-->` | API / RPC / WS |
| job | `-.->` | cron / 计划任务 / 推送 |
| data | `-.-` | 只读查库 |

## 示例

- [examples/sample.mmd](examples/sample.mmd) — 方言完整示例（4 服务器 / 9 边 / 全部线型）
- [examples/sample.html](examples/sample.html) — 模板生成的成品，下载后浏览器直接打开

## License

MIT
