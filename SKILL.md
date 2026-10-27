---
name: cad-dev-framework
description: |
  AutoCAD 二次开发（AutoLISP/VLISP/DCL）领域方法论与工具箱。基于 15 份本地 CAD 开发书籍资料（9 份主源）的深度调研与两轮交叉校审，提炼 5 个核心心智模型、10 条决策启发式、4 类代码模式库与避坑清单。
  用途：作为 CAD 脚本开发顾问，提供从需求到交付的完整开发流程、代码模板、调试方法与部署方案。
  当用户提到「CAD 二次开发」「AutoLISP」「VLISP」「DCL」「写个 LISP 程序」「CAD 插件」「CAD 脚本」等触发。即使用户只是说「帮我写个改图层的程序」「这个 DCL 报错怎么解决」「用 CAD 开发框架看看」也应触发。
  不在一般性 CAD 使用问题（如绘图操作、命令行技巧）上自动触发。
---

# CAD 二次开发 · 思维工具箱

> 核心原则：先让核心逻辑跑通，再套交互壳；一切修改都走数据表，一切分发都声明版本。

## 框架概览

本框架是 AutoCAD AutoLISP/VLISP/DCL 二次开发的领域共识汇总，覆盖：开发流程（先想后写）、语言核心（函数体系与图元数据）、DCL 对话框、实战代码模式、避坑清单。所有内容均标注来源与推断边界，流派分歧如实呈现。

## 回答工作流（Agentic Protocol）

**核心原则：不凭记忆编造 CAD 函数行为，涉及具体函数/组码/版本行为时先核对知识库（references/research/）与官方帮助。**

### Step 1: 问题分类

| 类型 | 特征 | 行动 |
|------|------|------|
| 写新程序 | 用户描述需求，要求生成 AutoLISP/DCL 代码 | → 按「开发流程」组织：先定核心算法，再定交互方式 |
| 查错/调试 | 报错信息、程序不工作 | → 先按避坑清单逐项排查（禁调函数/转换遗漏/路径转义/时序） |
| 改现有程序 | 已有代码要加功能 | → 用图元数据表驱动模型定位组码与函数链 |
| 知识咨询 | 函数语法、DCL 属性、加载方式 | → 直接引用调研文档，标注来源；不确定的标【推断】 |

### Step 2: 开发前核对（写代码前必查）

**⚠️ 写任何 DCL 或涉及对话框的代码前，必须核对以下清单：**
- 对话框时序：`load_dialog → new_dialog → (set_tile/mode_tile/action_tile…) → start_dialog → done_dialog → unload_dialog`
- 对话框激活期禁调 28 个函数（command/entmake/ssget 交互/getpoint 等，完整列表见调研 03 §5）——图形操作一律放 done_dialog 之后
- DCL 语法：对象与属性小写、属性赋值以 `;` 结尾、`key` 唯一、至少一个 OK/Cancel 按钮
- `set_tile`/`get_tile` 字符串转换（整数 itoa、实数 rtos、回转 atoi/atof）
- 路径反斜杠转义 `\\`
- `command` 前 `(setvar "cmdecho" 0)`、结束恢复
- VLA 函数必须先 `(vl-load-com)`

### Step 3: 按心智模型输出

基于 Step 2 的核对结果，运用下方心智模型、代码模式库与决策启发式输出可加载测试的代码，并附：加载方式、最低版本声明、已知坑提醒。

## 核心心智模型（领域共识）

### 模型 1: 界面/输入与核心逻辑分离
**一句话**：先写核心算法（纯 LISP 跑通），再考虑交互输入方式（getpoint/getstring 或 DCL），二者不混写。
**证据**：铁道版给出 DCL 优先与 AutoLISP 优先两种思考法，共性都是"界面/数据输入与核心逻辑分离"（调研 01 §1.1）；入门教材的参数化设计（图形尺寸用变量表达）同源（调研 04 §2.2）。
**应用**：任何新需求先问"核心逻辑是什么、输入输出是什么"，先让纯 LISP 函数可独立测试，再套对话框。
**局限**：交互密集型程序（需要反复取点绘图）强行分离会增加上下文传递成本，此时命令式交互更直接。

### 模型 2: 图元数据表驱动
**一句话**：AutoCAD 中一切图元都是组码关联表，改图元 = `entget → subst → entmod（→ entupd 刷新）`。
**证据**：cth 改字高走"entget→subst 组码 40→entmod"（调研 02 §2.5）；stl 改直线端点组码 10/11（调研 04 §1.1）；DCL 中 key 是 LISP 与 DCL 的唯一桥梁（调研 03 §2.3）；系统变量 getvar/setvar 按名称取值/设值、配合 set_tile/get_tile 预设与回写（调研 03 §4 dia5a 骨架）。
**应用**：面对"改任何图元属性"类需求，第一反应是查组码表（40 字高、8 图层、10/11 端点、0 类型、5 句柄），而不是想命令。
**局限**：块内嵌套对象、复元体（polyline 等）需要 entupd 甚至 regen 才显示更新；纯表驱动代码可读性低于 VLA 对象方式。

### 模型 3: 命令入口 + 子函数模块化
**一句话**：`(defun c:xxx () ...)` 定义命令入口，内部逻辑拆普通 defun 子函数；局部变量在形参表 `/` 后声明。
**证据**：四例全部 `c:` 前缀命令（调研 01 §1.2）；dia5a/viewdcl 主命令+子函数分层（调研 01 §1.2、03 §4）；局部变量避免污染全局命名空间（调研 01 最佳实践 #4）。
**应用**：所有程序都按"命令入口 / 对话框控制 / 数据子程序"三层组织，便于单测与复用。
**局限**：过度拆分会增加调用链深度，小程序无需强拆。

### 模型 4: DCL 是被动壳
**一句话**：DCL 只描述界面，控制权全在 AutoLISP；对话框激活期间禁止调用 28 个图形函数，图形操作一律放 done_dialog 之后。
**证据**：DCL 被动、key 为桥梁（调研 03 §1.1）；第 15 章明确禁调清单（调研 03 §5、05 §1.4）；dia5a 完整时序链（调研 03 §4）。
**应用**：设计对话框程序时先画"对话框内能做什么（控件交互/取值）、对话框外能做什么（图形操作）"的边界。
**局限**：部分需求（实时预览图形效果）在对话框内无法实现，需采用"对话框设置→关闭→执行→可再次打开"模式。

### 模型 5: 版本兼容与部署意识
**一句话**：程序从编写第一天就要考虑"在哪个 CAD 版本、用什么加载方式、分发给谁"。
**证据**：四例标注"适用 AutoCAD 10.0 以上（R13 调试通过）"（调研 01 §1.4、04 §1.1、05 §1.3）；acadlspasdoc 控制 acad.lsp 加载范围（调研 01 §1.4、05 §1.3）；FAS/VLX 打包分发避免源码暴露（调研 01 §1.4）。
**应用**：交付时附加载说明（acad.lsp / APPLOAD / load）+ 最低版本 + 依赖（vl-load-com）。
**局限**：不同版本系统变量、函数行为有差异，资料时代（R13/2000）与现版本需以官方帮助实测为准。

## 决策启发式

1. **改图元属性**：`entget` → `subst` 替换目标组码 → `entmod` →（复杂图元）`entupd` 刷新。【推断】entupd 为通用最佳实践（四例源码 cth 未使用）
2. **做对话框**：先 VIEWDCL.LSP 预览 DCL 布局，再写 LISP 控制代码，最后 `(load)` 联调，每层错误域独立排查【推断，调研 01 §1.3】
3. **对话框激活期**：禁调 command/entmake/ssget(交互)/getpoint 等 28 个函数，图形操作放 `done_dialog` 之后
4. **DCL 取值/设值**：`set_tile`/`get_tile` 只走字符串——整数 `itoa`、实数 `rtos`、回转 `atoi`/`atof`
5. **路径字符串**：反斜杠必须转义为 `\\`（`(load "D:\\lisp\\cth")`），load_dialog 无路径时按四层顺序搜寻（当前工作路径→图形文件所在→支持路径→CAD 程序目录），失败返回 -1
6. **用 command**：先 `(setvar "cmdecho" 0)` 抑制回显，结束恢复；命令函数以 `(princ)` 收尾
7. **写 DCL**：对象与属性小写、赋值以 `;` 结尾、`key` 唯一且大小写敏感、至少一个 OK/Cancel；`list` 多项用 `\n` 分隔
8. **选绘图方式**：简单交互（少量画图）用 `command`；批量/稳定用 `entmake` 或 VLA；追求可读性用 VLA（先 `(vl-load-com)`）
9. **遍历选择集**：`ssget` 尽量带过滤表 `'((0 . "TEXT"))`；遍历用 `repeat` + `ssname` + `sslength`
10. **分发程序**：提供"源码 LSP + 打包 VLX"双通道；常驻工具放 acad.lsp/acaddoc.lsp，低频命令用 APPLOAD；声明最低版本与依赖【推断，调研 01 §1.4】

## 流派对比（分歧如实呈现）

| 分歧 | 派系 A | 派系 B | 处理建议 |
|------|--------|--------|---------|
| 绘图路线 | `command` 直观（四例实战大量使用） | `entmake`/ActiveX 稳定快（Visual LISP 教程推荐） | 双轨提供：简单交互→command；批量稳定→entmake/VLA |
| acad.lsp 加载 | 0：仅任务打开的第一个图形加载（传统，兼顾性能） | 1：每个文档都加载（命令处处可用） | 按部署需求选；常驻工具用 acaddoc.lsp 更可控 |
| 思考法起点 | DCL 优先（先界面后逻辑） | AutoLISP 优先（先逻辑后界面） | 新手走 LISP 优先；界面复杂需求走 DCL 优先，共性都是逻辑/界面分离 |

## 代码模式库

### 模式 A: 遍历选择集并修改图元（批量修改）
```lisp
(defun c:cth ( / ss i en ed new)
  (setq ss (ssget '((0 . "TEXT"))))          ; 带过滤表，避免误选
  (setq i 0)
  (repeat (sslength ss)
    (setq en (ssname ss i))
    (setq ed (entget en))
    (setq ed (subst (cons 40 5.0) (assoc 40 ed) ed))  ; 改字高为5
    (entmod ed)
    (setq i (1+ i)))
  (princ))
```
> 注：cth 程序同型（调研 02 §2.5/2.6）；复杂图元可补 `entupd` 刷新显示。

### 模式 B: 交互取点 + polar 参数化绘图
```lisp
(setq p1 (getpoint "\n指定起点: "))
(setq p2 (getpoint p1 "\n指定终点: "))
(setq ang (angle p1 p2) len (distance p1 p2))
(setq p3 (polar p1 ang (/ len 2)))           ; 中点
(command "line" p1 p2 "")
```

### 模式 C: entmake 直接建图元
```lisp
(entmake (list '(0 . "LINE") (cons 10 p1) (cons 11 p2)))
```

### 模式 D: VLA 对象操作
```lisp
(vl-load-com)
(setq doc (vla-get-ActiveDocument (vlax-get-acad-object)))
(setq ms (vla-get-ModelSpace doc))
(vla-addCircle ms (vlax-3d-point p) 5.0)
```

### 模式 E: DCL 对话框标准骨架（dia5a 型）
```lisp
(defun c:dia5a ()                 ; 主命令
  (setvar "cmdecho" 0)
  (dcl_dia5a)                     ; 调用对话框子程序
  (princ))

(defun dcl_dia5a ()               ; 对话框控制
  (setq dcl_id (load_dialog "dia5a"))
  (new_dialog "dia5a" dcl_id)
  (set_tile "edit_p" (rtos (getvar "plinewid")))  ; 预设值：getvar→rtos→set_tile
  (set_tile "edit_s" (itoa (getvar "savetime")))
  (action_tile "accept" "(ok_dia5a)(done_dialog 1)")  ; 动作拼成一个字符串
  (start_dialog)
  (unload_dialog dcl_id))

(defun ok_dia5a ()                ; 数据回写
  (setvar "plinewid" (atof (get_tile "edit_p")))
  (setvar "savetime" (atoi (get_tile "edit_s"))))
```

## 避坑清单（高频）

1. DCL 激活期禁调 28 个函数 → 对话框强制终止报"AutoCAD 拒绝的函数"
2. `new_dialog` 后漏 `start_dialog` → 对话框失控
3. `mode_tile` 0/1 反直觉（0=恢复作用、1=禁用）
4. `action_tile` 多表达式必须拼成一个字符串；内嵌字符串变量要 `\"` 转义
5. `set_tile` 传非字符串值
6. `get_attr` key/属性名大小写敏感，不一致返回 ""
7. DCL 语法错误写入工作目录 ACAD.DCE，用记事本打开查
8. 初学者严禁修改官方 BASE.DCL
9. 未经测试就把程序加入 acad.lsp 自动加载【推断，调研 01 §1.4 加载机制引申】
10. 路径反斜杠不转义（`D:\lisp` 应为 `D:\\lisp`）
11. `vlax-curve-getClosestPointTo`：CHM 目录标题误作 isClosePointTo，以正文为准

## 诚实边界

此 Skill 基于本地书籍资料（R13/2000 时代编写）提炼，存在以下局限：
- 组码/系统变量未全量成体系（附录 B/C 未逐条收录），复杂场景需查 AutoCAD 官方帮助
- 《AutoLisp Vlisp 函数.chm》个别页标题与正文不一致，以正文为准
- 部分条目（坐标系/句柄行为/版本行为）为函数语义推断，落地前建议对照官方帮助核实
- 现代 CAD 版本（2004+）行为差异需实测验证，资料未覆盖
- 调研时间：2026-10；调研文档见 `references/research/`（01 方法论 / 02 语言核心 / 03 DCL / 04 实战模式 / 05 坑与工具链）

## 附录：调研来源

调研过程详见 `references/research/` 目录，经两轮独立校审（主视角通读 + 对照原著逐条核验）后修复 10 项阻断级问题。

### 主源资料（本地语料，最高权重）
- 《AUTOLISP@DCL基础篇(全)》铁道出版社（推荐核心）
- 《AutoCAD Visual LISP教程.pdf》
- 《AUTOLISP语言参考.pdf》
- 《autolisp实用编程四例.doc》（cth/stl/xlin/atxt 源码）
- 《CAD-lisp-实用小程序(源代码).pdf》（TX/TC/ZG）
- 《AutoLisp Vlisp 函数.chm》、《DCL控件属性.docx/.xlsx》、《Base.dcl an Acad.dcl.docx》、《系统变量.docx》

### 弃用资料
- 《Autolisp基础教程.pdf》《AutoLISP从入门到精通初级教程.doc》《Visual--Lisp-编程基础.ppt》《LISP语言(马希文).pdf》为辅助；《AutoLisp学习教程.chm》乱码不可读

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
