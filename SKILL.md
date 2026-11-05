---
name: arch-diagram
description: 服务架构图双向转换。方法一 nl2mmd：自然语言描述的服务器/服务/子服务/调度关系 → 约定方言的 Mermaid；方法二 mmd2html：该方言 Mermaid → 终端风交互 HTML（可拖拽、收合上浮、Alt 全显把手、localStorage 记忆布局）
---

# arch-diagram

两个方法，可单独用，也可串联（自然语言 → Mermaid → HTML）。

- **nl2mmd**：自然语言 → Mermaid 方言
- **mmd2html**：Mermaid 方言 → 交互式 HTML（模板：本 skill 目录 `assets/template.html`）

## 共同约定：Mermaid 方言

结构（三层，与 HTML 的 服务器→服务→子服务 对应）：

- 顶层 `subgraph` = **服务器**，内写 `direction TB`
- 二层 `subgraph` = **服务**（仅当有子服务时才建子图；无子服务的服务直接写成节点）
- 服务子图内的**节点** = 子服务
- 所有标题用带引号文本：`subgraph X["标题"]`、`N["标题"]`

**备注（meta）**：标题内用 ` | ` 分隔名称与备注，生成 HTML 后显示在名称旁的浅色小字（服务器规格、端口、任务量、参数等）。无备注就不写 ` | `：

- `subgraph WEB["Web 边缘 | 双活 · LAN"]` → 服务器名 "Web 边缘"，备注 "双活 · LAN"
- `WEB_GW["API 网关 | QPS 3k"]` → 服务名 + 备注
- 解析规则：取**最后一个** ` | `，左侧为名称、右侧为备注

边类型由**线型**表达，固定不变：

| 类型 | 线型 | 语义 |
|---|---|---|
| sync 同步 | `==>` | 数据同步 / 写回 |
| call 调用 | `-->` | API / RPC / WS |
| job 定时 | `-.->` | cron / 计划任务 / 推送 |
| data 读取 | `-.-` | 只读查库 |

- 边标签：`==>|"type · 时刻或频率"|`，如 `==>|"sync · 07:10 4×/日"|`；`-->|call|` 可省略说明
- 节点 id 用大写蛇形：`<服务器>_<服务>_<子服务>`，如 `NAS_SCHED_FLAT`
- 边可以锚在子图上（服务级/服务器级），代表该边在该粒度
- `flowchart LR` 开头；同宿主的内部边照常写（生成 HTML 时会画成右出右入的环线）

## 方法一：nl2mmd（自然语言 → Mermaid）

1. **抽实体**：机器/主机/环境 → 服务器；其上安装或运行的软件 → 服务；软件内的组件/功能/任务/库 → 子服务。
2. **抽关系**：动词映射类型——同步/写回/镜像 → sync；调用/请求/收发 → call；每日/定时/cron/xx:xx/推送 → job；读取/查询/白盒 → data。时间与频率写进边标签。
3. **层级拿不准就放服务级**，不要虚构子服务。
4. 数字与规格信息（端口、规格、任务量、IP 段等）进节点备注：`["名称 | 备注"]`；时间与频率进边标签。不要丢。
5. 输出一个完整 `flowchart LR` 代码块，无需解释。

## 方法二：mmd2html（Mermaid → 交互 HTML）

### 步骤

1. 读 `assets/template.html` 全文（约 38KB，单文件自包含，默认数据即 `examples/sample.mmd` 的内容）。
2. 解析 Mermaid（按上方方言），建立 `mermaid id → 层级路径` 映射：路径为点连接的 `服务器.服务.子服务`（按嵌套深度截断）。边锚在子图时，路径停在对应层级。
3. 把模板中 `/*==DATA==*/` 到 `/*==DATA-END==*/` 之间的**整块**替换为新数据块，其余代码一字不改：

```js
/*==DATA==*/
var ARCH_TITLE='图标题（终端标题栏与 standalone <title>）';
var ARCH_LSKEY='arch-<slug>';   // localStorage 键，每张图唯一
var TYPE={sync:{c:'#3ecf8e',dash:'10 4',dur:'1.5s',label:'同步'},
 call:{c:'#58a6ff',dash:'6 5',dur:'1.1s',label:'调用'},
 job :{c:'#f0a942',dash:'2 4',dur:'0.85s',label:'定时'},
 data:{c:'#b18cff',dash:'14 5',dur:'1.9s',label:'读取'}};
var COLS=[['srvA','srvB'],['srvC']];   // 默认列布局：数组每列 = 一组服务器 id
var SERVERS=[
 {id:'srvA',short:'A',name:'A · 显示名',meta:'备注',ac:'#b18cff',svc:[
   {id:'svc1',short:'svc1',name:'服务1',meta:'',subs:[
     {id:'sub1',short:'sub1',name:'子服务1',meta:''}]},
   {id:'svc2',short:'svc2',name:'服务2'}]},
];
var EDGES=[
 {from:'srvA.svc1.sub1',to:'srvC.svc2',type:'sync',sched:'07:10 4×/日'},
];
/*==DATA-END==*/
```

字段规则：

- `id` 用 ASCII（点路径由此拼出）；`name` 可中文；`short` 是连线/日志里的短名
- `meta` 来自节点/子图标题里最后一个 ` | ` 之后的部分，没有则省略
- `ac` 从色板轮换：`#b18cff #3ecf8e #39d0c4 #58a6ff #f0a942`
- `type` ∈ `sync|call|job|data`（对应 Mermaid 线型）；`sched` 来自边标签里 type 之后的部分，没有就写 `实时`
- `COLS`：按边的流向粗排（源在左、汇在右）；单服务器的列生成器会自动垂直居中

4. 产物写到目标路径（fragment 版）。需要双击可用时，包一层：

```html
<!doctype html><html lang="zh-CN"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{ARCH_TITLE}</title>
<style>body{margin:12px;background:#080b12}</style></head><body>
{fragment}</body></html>
```

### QA

1. **必做**：抽出 `<script>` 跑 `node --check`；核对数据——服务器/服务/子服务数与 Mermaid 一致，边数 = Mermaid 边数，默认视图连线数 = 去重后 (from,to,type) 组数。
2. **有浏览器时**：起本地服务**必须带 charset 头**（`python3 -m http.server` 不带 charset 会中文乱码，需自定义 handler 给 `.html` 加 `Content-Type: text/html; charset=utf-8`），打开检查标题、计数、`▾ servers` 粒度切换、点击收合。
3. 模板只改 DATA 块；其余代码（拖拽/收合上浮/持久化）不要重复实现。

## 示例

- `examples/sample.mmd` — 方言完整示例（4 服务器、9 条边、覆盖全部 4 种线型与同宿主边）
- `examples/sample.html` — 由模板生成的成品（双击可用）

## 生成后的 HTML 自带能力（不要在模板外重复实现）

拖拽移动（框顶把手，按住 Alt 全显）、右缘调宽、收合上浮、三种粒度预设（servers/services/leaves）、edge log / inspector / session log、布局与收合状态存 localStorage（键 = ARCH_LSKEY）。
- 内置编辑：按住 Alt 单击任意节点头部（三层均可）改名称/备注/服务器框色、新增下属、删除（两段确认，级联子树与边）；Alt 单击连线可改类型/时刻或删边；Alt 按住时右上出现 ⇱ export，弹窗展示最新状态的 Mermaid（语法高亮 + 复制），内含 ⟲ relayout 与 ↺ 恢复默认数据。编辑结果随 localStorage 持久化。
