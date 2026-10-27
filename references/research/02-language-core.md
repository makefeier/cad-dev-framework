---
AIGC: Label="1"
---

# 02 AutoLISP / VLISP 语言核心

> 标注规则：【来源:文件名】= 书中明确说明；【推断】= 归纳延伸。
> 主要依据：《AUTOLISP语言参考.pdf》《AutoLisp Vlisp 函数.chm》《Visual--Lisp-编程基础.ppt》《autolisp实用编程四例.doc》《CAD-lisp-实用小程序(源代码).pdf》《AutoCAD Visual LISP教程.pdf》《AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf》

---

## 1. 语言本质

【来源: Visual--Lisp-编程基础.ppt】AutoLISP 是 **解释型、无类型、表处理** 语言：
- 程序即数据（S-表达式），一切以表 `(...)` 形式存在；
- 变量无类型声明，运行时自动确定（整数/实数/字符串/表/图元名等）；
- 前缀表达式：运算符在前，如 `(+ 1 2)`。

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】基础语法规则要点（入门教材亦同）：
- 注释用分号 `;`；
- 表达式用小括号包裹，括号必须成对；
- 字符串用双引号；
- 函数调用即 `(函数名 参数...)`。

## 2. 常用函数体系（按用途分类）

> 【来源: AUTOLISP语言参考.pdf】提供按字母序的完整 AutoLISP 函数参考（权威语法来源）；《AutoLisp Vlisp 函数.chm》为 VLISP 函数手册（官方帮助中文版，含 vlax-* 系列）。

### 2.1 基本运算与赋值
- 算术：`+ - * /`、`1+`、`1-`、`abs`、`sqrt`、`expt`、`max`、`min`、`rem`
  - 例【来源: AutoLisp Vlisp 函数.chm】`(abs -99.25) → 99.25`
- 赋值：`setq`（核心）、`set`
- 求值控制：`quote '`、`eval`、`apply`、`mapcar`

### 2.2 表处理（List）
- 构造：`list`、`cons`、`append`、`(list ...)` 
- 取值：`car`、`cdr`、`cadr`、`caddr`、`nth`、`last`
- 操作：`subst`（替换元素）、`assoc`（按关键值查表）、`reverse`、`length`、`member`
- 【推断】图元数据（entget 返回）就是"点对表 + 关联表"结构，`assoc`+`subst`+`cons` 是修改图元的标配三件套。

### 2.3 字符串处理
- `strcat`（拼接）、`substr`（截取）、`strlen`、`strcase`（大小写）
- 数字↔字符串：`itoa`（整数转串）、`atoi`（串转整数）、`rtos`（实数转串，可带精度模式）、`atof`（串转实数）
- 【来源: autolisp实用编程四例.doc】xlin 程序用 `getreal` 取 x/y 坐标，再以 `(command (list x y))` 逐点连线；atxt 程序用 `(polar ...)` 沿弧线取点后以 `command "text"` 逐个写文字。

### 2.4 几何计算
- 点/距离/角度：`getpoint`、`getdist`、`getangle`、`getorient`、`distance`、`angle`、`polar`（极坐标求点）
- 转换：`trans`（坐标系转换）、`osnap`（捕捉点）
- 例【来源: autolisp实用编程四例.doc】stl 延伸直线用 `(angle p1 p2)` 求方向、`(polar p1 ang len)` 计算新端点；atxt 绕圆弧写文字用 `(polar ...)` 沿弧线取点。
- 【来源: AutoLisp Vlisp 函数.chm】VLISP 扩展：`(vlax-curve-getArea curve-obj)` 返回曲线面积，`vlax-curve-getDistAtParam`、`vlax-curve-getClosestPointTo`（返回曲线上离给定点最近的点）、`vlax-curve-getPointAtParam` 等曲线查询系列；失败返回 nil。（注：CHM 目录标题误作 isClosePointTo，正文语法为 getClosestPointTo，以正文为准，见 05 文档 §4）

### 2.5 图元操作
- 取得/修改：`entget`（取图元数据表）、`entmod`（按新数据表更新）、`entupd`（更新复元体显示）、`entdel`（删除/恢复）、`entmake`（新建图元）、`entlast`（取最后图元）
- 例【来源: autolisp实用编程四例.doc】cth 改文本高度：`entget` → `subst` 替换组码 40（字高）→ `entmod`；stl 同理改直线端点（组码 10/11）。完整遍历代码见 §2.6 模式 A。【推断】`entmod` 后调用 `entupd` 刷新显示为通用最佳实践，四例源码未使用。
- 例【来源: CAD-lisp-实用小程序(源代码).pdf】VLISP 对象方式：`(vla-get-ModelSpace ...)` 取模型空间、`vla-addCircle`/`vla-put-Layer`/`vla-put-Height` 等 VLA 方法修改对象属性。
- 【来源: AutoCAD Visual LISP教程.pdf】三种绘图方式对比：
  - `command`：最直观，直接调用 CAD 命令，但速度慢、受命令回显/版本影响；
  - `entmake`：直接写图元数据库，快、稳定，但需手写组码表；
  - **ActiveX（VLA）**：面向对象，代码可读性好，功能最强，是 VLISP 推荐方向。

### 2.6 选择集
- `ssget`（交互/条件选择）、`ssname`（取第 n 个成员）、`sslength`（成员数）、`ssadd`/`ssdel`/`ssmemb`
- 遍历模式【推断】：`(setq i 0) (repeat (sslength ss) (setq en (ssname ss i)) ... (setq i (1+ i)))`（cth 程序即此模式，见 §2.5 与模式 A）

### 2.7 文件 I/O
- 打开/关闭：`open`、`close`
- 读写：`read-line`、`write-line`、`read`、`princ`/`print`/`prin1`（写文件或命令行）
- 文件对话框：`getfiled`（浏览选择文件）
- 【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】文件 I/O 是基础篇核心章节之一；VIEWDCL 程序用 `(getfiled "打开DCL文件" "dcl" 2)` 选取 .dcl 文件。

### 2.8 流程控制
- 分支：`if`、`cond`、`when`（VLISP）
- 循环：`while`、`repeat`、`foreach`
- 例【来源: Visual--Lisp-编程基础.ppt】分支循环是入门必修：`(if (> x 0) ...)`、`(repeat n ...)`、`(while (< i n) ...)`。

## 3. VLISP 扩展能力（vlax-* / vla-*）

【来源: CAD-lisp-实用小程序(源代码).pdf】VLISP 程序必须先用 `(vl-load-com)` 加载 ActiveX 支持，随后可：
- 获取对象：`(vla-get-ModelSpace (vla-get-ActiveDocument (vlax-get-acad-object)))`
- 新建对象：`vla-addLine`、`vla-addCircle`、`vla-addText` 等
- 修改属性：`vla-put-*`（如 `vla-put-Layer`、`vla-put-Height`、`vla-put-Color`）
- 查询：`vla-get-*`
- 通用辅助：`vlax-get-property`、`vlax-put-property`、`vlax-invoke-method`

【来源: AutoLisp Vlisp 函数.chm】曲线类扩展函数 `vlax-curve-*`（getArea/getDistAtParam/getClosestPointTo/getPointAtParam 等）是二次开发做测量、标注、路径计算的重要工具。

【推断】VLISP 扩展覆盖：文件系统（vl-file-*）、注册表（vl-registry-*）、目录操作（vl-directory-*）、字符串扩展（vl-string-*）——均可作为 Skill 的进阶能力模块。

## 4. 典型代码模式（语言层）

### 模式 A：遍历选择集并修改图元
```lisp
(defun c:cth ( / ss i en ed new)
  (setq ss (ssget '((0 . "TEXT"))))          ; 只选文本
  (setq i 0)
  (repeat (sslength ss)
    (setq en (ssname ss i))
    (setq ed (entget en))
    (setq ed (subst (cons 40 5.0) (assoc 40 ed) ed))  ; 改字高为5
    (entmod ed)
    (setq i (1+ i)))
  (princ))
```
【来源: autolisp实用编程四例.doc】（cth 程序同型）

### 模式 B：交互取点 + polar 参数化绘图
```lisp
(setq p1 (getpoint "\n指定起点: "))
(setq p2 (getpoint p1 "\n指定终点: "))
(setq ang (angle p1 p2) len (distance p1 p2))
(setq p3 (polar p1 ang (/ len 2)))           ; 中点
(command "line" p1 p2 "")
```
【来源: autolisp实用编程四例.doc】（xlin/stl 同型）

### 模式 C：entmake 直接建图元
```lisp
(entmake (list '(0 . "LINE") (cons 10 p1) (cons 11 p2)))
```
【来源: AutoCAD Visual LISP教程.pdf】（entmake 方式示例）

### 模式 D：VLA 对象操作
```lisp
(vl-load-com)
(setq doc (vla-get-ActiveDocument (vlax-get-acad-object)))
(setq ms (vla-get-ModelSpace doc))
(vla-addCircle ms (vlax-3d-point p) 5.0)
```
【来源: CAD-lisp-实用小程序(源代码).pdf】（TX/TC/ZG 程序同型）

### 模式 E：表 + mapcar 批量处理
```lisp
(mapcar '(lambda (x) (command "erase" x "")) lst)
```
【推断】（基于 Visual--Lisp-编程基础.ppt 的 mapcar 教学与 DCL 篇批量取值的组合归纳）

## 5. 函数速查索引建议（供 Skill 蒸馏）

【推断】Skill 内置函数库按"高频必备 / 常用 / 扩展"分层：
- 高频：setq car cdr cons list assoc subst entget entmod entmake command ssget ssname sslength getpoint getdist getangle polar distance angle strcat rtos itoa atoi atof substr if cond while repeat princ mapcar
- 常用：entupd entdel entlast nentsel osnap trans getkword getfiled open read-line write-line close vl-load-com vla-* vlax-*
- 扩展（参考手册查询）：vlax-curve-*、vl-registry-*、vl-directory-*、vl-file-*、vl-string-*、grread、menucmd
*（内容由AI生成，仅供参考）*
*（内容由AI生成，仅供参考）*
