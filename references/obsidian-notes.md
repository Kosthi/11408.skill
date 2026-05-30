# 写入 Obsidian 笔记库（27Kaoyan）

> 把用户**显式要求**整理的内容沉淀成笔记、写进用户的 Obsidian 考研库时，按本文执行。
> 本库已按"frontmatter 属性 + MOC + 查询视图"重构，新建笔记只要 frontmatter 写对，导航/错题本/考纲覆盖会**自动收录**——不要手动去改任何索引或总览页。

## 1. 何时写 / 不写

- **写**：用户明确说"做个笔记 / 记到笔记里 / 整理成笔记 / 存到 Obsidian / 加到考研库 / 这道错题收一下"。
- **不写**：普通问答、解题、诊断——除非用户开口要存，否则只在对话里回答，不碰仓库。
- 写之前先把内容讲清楚/算对，再落库；落库是把已确认的内容沉淀，不是边想边写。

## 2. 仓库与访问方式（关键坑）

- vault 名 **27Kaoyan**，在 iCloud：`/Users/koschei/Library/Mobile Documents/iCloud~md~obsidian/Documents/27Kaoyan`。
- **内置 Read 工具 / `cat` / `find` 读这个 iCloud 路径会 `EPERM`**。只能用 `obsidian` CLI（需 Obsidian 在运行且该 vault 已打开）：
  - 读：`obsidian read vault="27Kaoyan" path="<相对路径>"`（含中文/空格用双引号包整个值）
  - 列文件/元数据/写入：`obsidian eval vault="27Kaoyan" code="..."`
- 若 CLI 报 **"Vault not found"**：vault 没打开。先 `open "obsidian://open?vault=27Kaoyan"`，等几秒再重试（用 `obsidian eval ... code="app.vault.getName()"` 确认）。
- 环境备注：Obsidian 1.12.7；**Bases 核心插件**已启用；**Dataview**已装并启用、`enableDataviewJs=true`。本机 Bash 安全分类器偶发不可用（只读不受影响、写操作会被卡），稍等重试即可。

## 3. 放进哪个文件夹（文件夹只管存放）

| 内容 | 目录 |
|---|---|
| 高数 / 线代 / 概率 知识笔记 | `10_数学/11_高数/<专题>/`、`10_数学/12_线代/`、`10_数学/13_概率论/` |
| 408 四科知识笔记 | `20_408/21_数据结构/`、`22_计算机组成原理/`、`23_操作系统/`、`24_计算机网络/` |
| 数学错题 | `30_题目/数学错题/` |
| 408 错题 | `30_题目/408错题/` |
| 真题复盘 | `30_题目/真题复盘/` |
| 资料 / 模板 / 考纲清单 | `40_资料/`（模板在 `40_资料/_模板/`） |

入口路由页在 `00_入口/`，科目 MOC 是各 `…总览.md`——**这些都不用手动维护**。

## 4. frontmatter 模板（必须写对，否则视图收不进来）

```yaml
---
title: 可读标题（允许数学符号/LaTeX）
created: 2026-05-30        # YYYY-MM-DD，必填（不要用旧字段 date）
status: seed              # seed收集 / draft整理中 / done可复习 / review二轮重点 / archived已掌握或弃用
type: note                # note知识 / problem题目错题 / index索引MOC / method方法
subject: 高数              # 高数 / 线代 / 概率 / 计组 / 数据结构 / 操作系统 / 计网
exam_points:              # ★考点稳定 id 列表，见第 5 节；一篇可挂多个
  - gs-improper-01
tags:                     # ★只放"知识主题"，见下；不放 method/qtype/status/source
  - 高数/反常积分
method: []                # 可选：凑微分 / 换元 / 对称性 / ε吸收 …
qtype:                    # 可选（题目）：选择题 / 填空题 / 解答题
source:                   # 可选：来源
aliases: []               # 可选：旧名 / 符号名 / 常见搜索名
---
# H1 标题（与 title 同义即可）

正文…
```

规则：
- **tags 只放跨章节的知识主题**（如 `高数/曲线弧长`、`408/计组/数据表示与运算`），它不是第二套分类系统。分类靠 `subject`+`exam_points`。
- 旧标签命名空间一律不要再用：`数学分析/* 考研数学 math/* calculus/* 数学/* 方法论/* 题目收集` 等。
- `method / qtype / status / source` 走独立属性，**绝不进 tags**。
- 错题/题目 → `type: problem`（会自动进"错题本"视图）；带 `qtype`、`source` 更好。
- 模板在 `40_资料/_模板/知识笔记.md`，可先 `obsidian read` 取来照抄。

## 5. exam_points 怎么填

- 权威名单在 **`40_资料/考纲清单.md`**（一个 yaml 数据块，字段 `{id, subject, chapter, name, freq}`）。
- 按 subject 找最贴合的 `id` 填进 `exam_points`（如反常积分→`gs-improper-01`、乘法→`co-mult-01`）。一篇可挂多个。
- **笔记里只放 id，考点中文名只存在清单里**。考纲有变只改清单文件，不动笔记。
- 408 的 `ds-* / os-* / cn-*` 命名空间已预留，缺的考点按 `<科目码>-<主题>-NN` 规则在清单续写一行再引用。

## 6. 写入步骤（推荐：本地起草 → 文件系统推送，免转义）

不要把长 LaTeX/中文塞进 shell 字符串（转义易错）。改走"内容走文件、不走命令行"：

1. 用 Write 工具把完整笔记（frontmatter + 正文）写到本地临时文件，如 `/tmp/note.md`，自己审一遍。
2. 让 Obsidian 进程（有 iCloud 权限）读本地文件、写进 vault：

```bash
obsidian eval vault="27Kaoyan" code="
const fs=require('fs');
const c=fs.readFileSync('/tmp/note.md','utf8');
const p='10_数学/11_高数/反常积分/我的新笔记.md';      // vault 相对路径
const dir=p.split('/').slice(0,-1).join('/');
const ex=app.vault.getAbstractFileByPath(p);
(async()=>{
  if(dir && !app.vault.getAbstractFileByPath(dir)){ try{await app.vault.createFolder(dir);}catch(e){} }
  ex ? await app.vault.modify(ex,c) : await app.vault.create(p,c);
})();
'written '+p"
```

3. 用 `app.vault.create/modify`（Obsidian API）而非裸 `fs.writeFileSync` 写 vault，能让元数据缓存即时一致。create/modify 是异步、CLI 不等待——写完用 `obsidian read vault="27Kaoyan" path="<p>"` 读回确认。

## 7. 改名 / 删除 / 维护视图时的坑（来自整改实战）

- **改名/移动**用 `app.fileManager.renameFile(file, newPath)`：会自动更新所有 wikilink。
  - 它会弹"是否更新链接"模态卡住后续操作。批量改名前先 `app.vault.setConfig('alwaysUpdateLinks', true)`（或点模态"总是更新"），做完**改回原值**。
  - 一次只 `await` 一个 renameFile；连续 await 多个 + 模态会死锁。
- **文件名禁用** `\ / : * ? " < > |`。需要"主题:子题"分隔时用**全角 `：`**（title/H1 可读即可，三层不要求逐字一致）。
- **删除**用 `app.vault.trash(file, true)`（进系统回收站，可恢复）。删前确认零入链：扫 `app.metadataCache.resolvedLinks` 反查，没人链接才删。
- **视图**：D1–D3（已整理总表/复习看板/错题本）是 **Bases `.base`**（核心插件，无 JOIN）；D4 **考纲覆盖**是 **DataviewJS**（做"列出 0 笔记考点"的覆盖矩阵）。
  - Bases `.base` 校验：`obsidian base:query vault="27Kaoyan" path="40_资料/已整理总表.base" view="已整理总表" format=md`。`groupBy` 是对象 `{property, direction}`；多条件用结构化 `or:/not:`。
  - DataviewJS 坑：① 插件持久启用要用 `app.plugins.enablePluginAndSave('dataview')`（`enablePlugin` 不写盘、重启丢）；② `enableDataviewJs` 默认关，要 `settings.enableDataviewJs=true` + `saveData`；③ 纯 eval 里 `require('obsidian')` 不可用，解析就用普通 JS 正则；④ 代码块里别写裸三反引号，用 `String.fromCharCode(96).repeat(3)` 拼围栏，免打断 code fence。

## 8. 写完后的自检

- 笔记会**自动**出现在 `已整理总表.base`（type=note/problem）、`复习看板.base`（按 status）、错题进 `错题本.base`（type=problem）、考点进 `考纲覆盖.md` 矩阵——**不需要手动改任何索引/总览页**。
- 确认没引入坏链：`obsidian eval vault="27Kaoyan" code="Object.values(app.metadataCache.unresolvedLinks).every(o=>Object.keys(o).length===0)"` 应为 `true`。
- 想让用户看见就提示打开对应 `…总览` 或 `40_资料/考纲覆盖.md`。
