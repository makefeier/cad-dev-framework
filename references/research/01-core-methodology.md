---
AIGC: Label="1"
---

# 01 开发方法论：从需求到交付的完整开发流程

> 本文档提炼自 F:\pt文件夹\cad开发 下的 CAD 二次开发书籍资料。
> 标注规则：【来源:文件名】= 书中明确说明；【推断】= 基于资料归纳的延伸结论。
> 主要依据：《AutoCAD Visual LISP教程.pdf》《autolisp实用编程四例.doc》《AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf》《Autolisp基础教程.pdf》《CAD-lisp-实用小程序(源代码).pdf》《系统变量.docx》

---

## 1. 总体开发流程

### 1.1 脚本设计（先想后写）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】书中给出两种整合设计思考流程（以 DCL 对话框程序为例）：

- **DCL 优先思考法**：①构思功能与期望界面 → ②编写 DCL 文件并预览修改 → ③编写 AutoLISP 程序搭桥 → ④预设界面起始值 → ⑤取得界面信息、结束对话框、编写子程序 → ⑥成果验收。
- **AutoLISP 优先思考法**：①构思功能 → ②先写 AutoLISP 程序（确定 *.lsp 正常执行）→ ③编写配套 DCL 文件并预览 → ④为 LISP 与 DCL 搭桥 → ⑤用 DCL 界面替代 INPUT 输入数据 → ⑥成果验收。

【推断】两种方法的共性：**界面/数据输入与核心逻辑分离**；先让核心算法（纯 LISP）跑通，再套对话框，可显著降低调试难度。此模式对无对话框的纯命令程序同样适用——先写命令函数，再考虑交互输入方式（getpoint/getstring 或 DCL）。

【来源: Autolisp基础教程.pdf】入门教材强调"参数设计"思路：把图形中可变的量（尺寸、数量、位置）定义为参数，程序根据参数生成图形。这是脚本设计的起点。

### 1.2 命令定义（C: 前缀）

【来源: autolisp实用编程四例.doc】实用程序均以 `(defun c:xxx () ...)` 形式定义命令，用户在命令行直接输入 `xxx` 即可执行，无需括号。

【来源: AutoCAD Visual LISP教程.pdf】命令函数定义规范：
- 用 `defun` 定义 `c:命令名` 即成为 AutoCAD 内部命令；
- 命令函数体最后用 `(princ)` 或 `(prin1)` 收尾，防止返回 nil 在命令行打印；
- 内部子程序用普通 `defun` 命名（如 `defun dcl_dia5a ()`），主命令调用子程序，实现模块化；【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第16章 dia5a 案例（页292）即以 dcl_dia5a 为对话框控制子函数。

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】书中示例 `(defun c:viewdcl () ...)`、`(defun c:dia5a () ...)` 均遵循"主命令 + 子函数"分层结构。

### 1.3 调试方法

【来源: AutoCAD Visual LISP教程.pdf】Visual LISP 集成开发环境（VLISP IDE）提供完整调试手段：
- 在代码中设置**断点**，单步执行（Step Into / Step Over）；
- **监视窗口**（Watch Window）查看变量当前值；
- **跟踪**（Trace）调用过程；
- 在 VLISP 控制台（Console）直接输入表达式求值，逐段验证；
- 通过 `!变量名` 在命令行查看变量值（书中演示 gardenpath 示例的调试）。

【来源: Autolisp基础教程.pdf】解释型语言的调试方式是"改一处、加载一次、试一次"，命令 `(load "文件")` 重新加载即可看到修改效果，无需编译。

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】DCL 对话框调试有专门提示：
- DCL 加载语意有误时，AutoCAD 会弹警告，并把错误记录在工作文件夹内的 **ACAD.DCE** 文件中，用记事本打开可定位错误；
- 随书附赠 **VIEWDCL.LSP** 预览程序，可先预览对话框效果再写控制代码，是 DCL 开发的标配调试手段。

【推断】DCL 程序调试顺序应为：先用 VIEWDCL 验证 DCL 语法与布局 → 再写 LISP 控制代码 → 最后用 `(load)` 加载联调。每层错误域独立，避免混合排查。

### 1.4 加载与分发方式

【来源: autolisp实用编程四例.doc】明确给出多种加载方式：
1. **acad.lsp 自动加载**：把程序放入 acad.lsp 或追加到 acad.lsp，AutoCAD 启动时自动加载，命令直接可用；
2. **`(load "文件名")`** 手动加载，注意路径中的反斜杠必须转义（如 `(load "D:\\lisp\\cth")`）；
3. 使用 APPLOAD（加载应用程序）对话框选择文件加载。

【来源: 系统变量.docx】`acadlspasdoc` 系统变量控制 acad.lsp 的加载范围：
- 0：仅将 acad.lsp 加载到 AutoCAD 任务打开的第一个图形中（系统变量.docx 原文口径）；
- 1：每个文档都加载（保证每个文档都有自定义命令）。
【来源: autolisp实用编程四例.doc】书中说明程序"适用于 AutoCAD 10.0 以上的 DOS 和 Windows 版本"（R13 为调试环境，调试通过），即需考虑版本兼容性，不同 CAD 版本下加载机制与函数能力有差异。

【来源: AutoCAD Visual LISP教程.pdf】VLISP 生成的程序可通过"生成应用程序"（make application）打包为 **FAS/VLX** 文件分发，避免源码暴露，加载方式同 LSP（APPLOAD 或 load）。

【推断】Skill 化后的分发建议：提供"源码 LSP + 打包 VLX"双通道；对只读性工具类命令建议加入 acad.lsp 或 acaddoc.lsp 自动加载；对低频命令用 APPLOAD 按需加载。

---

## 2. 反复出现的最佳实践

| # | 最佳实践 | 出处 |
|---|---------|------|
| 1 | 命令函数体以 `(princ)` 收尾，避免命令行回显 nil | AutoCAD Visual LISP教程.pdf、autolisp实用编程四例.doc |
| 2 | 程序开头 `(setvar "cmdecho" 0)` 抑制 command 回显，结束恢复 | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf（dia5a 示例）、autolisp实用编程四例.doc |
| 3 | 用 `defun` 定义 `c:xxx` 作为命令入口，内部逻辑拆分子函数 | 多份资料一致 |
| 4 | 局部变量用 defun 形参表 `/` 后声明，避免污染全局命名空间 | AutoCAD Visual LISP教程.pdf |
| 5 | 使用 `(vl-load-com)` 后才可调用 VLA 对象函数 | CAD-lisp-实用小程序(源代码).pdf |
| 6 | 路径字符串反斜杠必须转义为 `\\` | autolisp实用编程四例.doc |
| 7 | DCL 设计先预览（VIEWDCL），再写控制代码 | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |
| 8 | 数据转换显式化：整数 `itoa` / 实数 `rtos` / 字符串转回 `atoi` `atof` | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf、Visual--Lisp-编程基础.ppt |
| 9 | `mapcar` 批量设置/获取多个 DCL 控件值（key_list + val_list） | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |
| 10 | 程序尽量保持"纯 LISP + 少量 command"结构，便于跨版本移植 | AutoCAD Visual LISP教程.pdf（三种绘图方式对比） |

## 3. 反复出现的禁忌

| # | 禁忌 | 出处 |
|---|------|------|
| 1 | DCL 对话框激活期间调用 `command`、`entmake`、`ssget`（交互式）、`getpoint` 等函数，会导致对话框被强制终止并报错"AutoCAD 拒绝的函数" | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf 第15章明确列表 |
| 2 | `new_dialog` 后漏写 `start_dialog`，对话框失控，AutoCAD 无法正常运作 | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |
| 3 | `action_tile` 的动作表达式写成多组参数（错误写法是把多个表达式当作多参数传入），必须把多个表达式拼接在**一个字符串**里 | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |
| 4 | 动作表达式字符串内嵌字符串变量时不加 `\"` 转义（如 `"(setq str1 "ABC")"` 是错的，应为 `"(setq str1 \"ABC\")"`） | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |
| 5 | `set_tile` 传非字符串值（必须先用 itoa/rtos 转换） | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |
| 6 | 未经测试就把程序加入 acad.lsp 自动加载，启动报错影响所有绘图工作 | 【推断】（依据 autolisp实用编程四例.doc 加载方式章节提示归纳） |
| 7 | 修改 BASE.DCL（初学者"轻举妄动"后果不堪设想） | AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf |

---

## 4. 需求→交付 Checklist（整理）

【推断】综合各资料，提炼适用于 Skill 蒸馏的交付流程：

1. 明确需求（改什么、批处理对象、输入输出）；
2. 设计参数与数据结构（图元类型、选择集、层名、文字样式）；
3. 编写核心算法（纯 LISP 函数，可脱离命令独立测试）；
4. 定义命令入口 `c:xxx`，设计用户交互（命令行 getxxx 或 DCL 对话框）；
5. 用 VLISP 调试（断点/监视/控制台）；
6. 试用加载 `(load "...")`，覆盖正常与异常输入；
7. 按需打包 FAS/VLX，编写加载说明（acad.lsp 或 APPLOAD）；
8. 标注版本兼容性（如"适用于 AutoCAD 10.0 以上，R13 调试通过"）与依赖（vl-load-com）。

> 矛盾保留：关于"是否每个文档都加载 acad.lsp"（acadlspasdoc=0/1），资料本身给出两种取舍，无定论；实际应以部署环境需求为准。另《AutoCAD Visual LISP教程.pdf》建议尽量用 entmake/ActiveX 替代 command（稳定、快），而《autolisp实用编程四例.doc》的实战程序大量使用 command——两书对 command 的态度存在差异，属不同年代/不同场景的取舍，保留。
*（内容由AI生成，仅供参考）*
*（内容由AI生成，仅供参考）*
