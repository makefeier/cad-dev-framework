---
AIGC: Label="1"
---

# 04 实战案例与代码模式

> 标注规则：【来源:文件名】= 书中明确说明；【推断】= 归纳延伸。
> 主要依据：《autolisp实用编程四例.doc》《CAD-lisp-实用小程序(源代码).pdf》《AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf》（第二篇 61 个实用范例）

---

## 1. 实战程序清单与解法拆解

### 1.1 《autolisp实用编程四例.doc》— 4 个经典程序

| 程序 | 功能 | 核心解法 |
|------|------|----------|
| cth | 修改文本高度 | 选择集筛选文本 → entget 图元表 → subst 替换组码 40 → entmod |
| stl | 延伸直线到指定长度 | entsel 选线 → angle 求方向 → polar 计算新端点 → entmod 更新端点组码（10/11） |
| xlin | 快速画线（连续画） | 循环 getreal 取 x、y 坐标 → (command (list x y)) 逐点成线，减少命令输入次数 |
| atxt | 绕圆弧写文本 | polar 沿弧线取多个点位 → command "text" 逐个写入，实现弧形文字 |

通用加载说明（书中明确）：程序需放入 acad.lsp 或通过 load 加载；路径反斜杠转义；适用于 AutoCAD 10.0 以上的 DOS 和 Windows 版本（R13 调试通过）。

【推断】`entmod` 后调用 `entupd` 刷新显示为通用最佳实践，四例源码（cth）未使用。

【推断】这四例代表最实用的四类需求：**批量属性修改（cth）、图元几何修正（stl）、绘图命令封装（xlin）、复杂几何布点（atxt）**——Skill 的"常用工具箱"应覆盖这四类。

### 1.2 《CAD-lisp-实用小程序(源代码).pdf》— 3 个 VLISP 源码

| 程序 | 功能 | 核心解法 |
|------|------|----------|
| TX | 圆自动同心 | vl-load-com → VLA 对象取圆心 → vla-addCircle 绘制同心圆 |
| TC | 图层快速修改 | 选择对象 → vla-put-Layer 批量改图层 |
| ZG | 字高自动修改 | 选择文本对象 → vla-put-Height 批量改字高 |

共同模式（书中明确）：全部使用 `(vl-load-com)` 开启 ActiveX，通过 VLA 对象操作模型空间（`vla-get-ModelSpace`），用 `vla-put-*` 写属性、`vla-add*` 建对象。

【推断】VLISP 时代的主流解法是 ActiveX 对象方式；与经典 entget/entmod 相比，VLA 方式代码短、面向对象、适合属性批量修改类工具。

## 2. 可复用代码模式（按场景分类）

### 2.1 批量处理模式
- 选择集遍历 + 逐对象修改（cth 模式）：`ssget → repeat → ssname → entget/entmod`
- VLA 批量：`vlax-for` 或循环选择集 + `vla-put-*`（TC/ZG 模式）
- 【推断】通用模板：先 `ssget` 按组码/类型过滤（`(0 . "TEXT")`、`(8 . "图层名")`），再做修改，最后 `entupd`/`vla-update` 刷新。

### 2.2 参数化绘图模式
- 交互取参：`getpoint/getdist/getangle/getint/getreal/getstring`
- 几何计算：`polar`（极坐标偏移）、`distance`、`angle`
- 绘制：`command`（直观）或 `entmake`（快速）或 VLA
- 例（xlin/stl/atxt）：以"起点+方向+长度"三步法画线，用 `polar` 代替手工坐标计算。
- 【来源: Autolisp基础教程.pdf】入门教材强调参数化设计：图形尺寸用变量表达，改参数即改图形。

### 2.3 图层/样式管理
- 读当前层：`(getvar "clayer")`
- 改对象图层：entmod 替换组码 8，或 `vla-put-Layer`
- 设置当前图层/系统变量：`(setvar "clayer" "新层")`
- 创建图层：`(command "layer" "new" "层名" "")` 或 entmake 图层表
- 【来源: CAD-lisp-实用小程序(源代码).pdf】TC 程序即改对象图层；【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第16章 dia5a 类程序常需切换当前层后绘图。

### 2.4 自动标注与文字
- 文字：command "text"、entmake 文本、VLA `vla-addText`
- 字高控制：`(getvar "textsize")` 或直接改组码 40 / `vla-put-Height`
- 数值转字符串后拼接标注内容：`(strcat "面积=" (rtos area 2 2))`
- 【来源: autolisp实用编程四例.doc】atxt 绕圆弧标注文字、cth 批量改字高即为标注类工具代表。

### 2.5 DCL 对话框程序模式（见 03 文档）
- 【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第16章完整案例 dia5a：系统变量预设 + 编辑框交互 + 确定回写，是"对话框式设置工具"的标准骨架；
- 【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第三篇第16~21章"DCL 新手上路"系列覆盖全部控件组合的完整可运行示例：16 编辑框/按钮/文字、17 切换开关/单选按钮、18 列表/下拉列表、19 图像/图像按钮、20 滑动条、21 结训测验（各章要点见 03 文档 §2.4）。

## 3. 范例库来源划分（可选题库）

### 3.1 第二篇 61 个实用范例（第11章，纯 AutoLISP，不含 DCL）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第二篇"AutoLISP 程序设计六十大进阶秘籍"第11章 11-1~11-61 为**纯 AutoLISP 绘图/对象范例**，不含 DCL 控件。覆盖方向（据目录与已读章节归纳）：
- 图形绘制类（凹形图、四圆孔、时钟、螺旋、砖墙等参数化图形）；
- 文字处理类（文字大小写转换、绕弧文字等）；
- 对象/图层管理类（统计图块、图层管理、xrecord 等）。

【推断】这类纯 LISP 范例是 Skill"绘图/对象操作模板"的来源库，与 DCL 无关，蒸馏时应归入语言核心/实战绘图模式（02/04 文档），而非对话框章节。

### 3.2 第三篇 DCL 新手上路系列（第16~21章，含 DCL）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第三篇第16~21章为 DCL 完整案例系列（各章要点见 03 文档 §2.4）：
- 系统变量/绘图默认值设定类（dia5a，第16章）；
- 词库/文字内容管理（列表框+单选钮组合，dia7，第18章）；
- 比例/小数点/格式设定（下拉列表，dia7c）；
- 开关控制（toggle）与数值微调（slider，dia6/dia9，第17/20章）；
- 图像菜单与颜色选择（image/image_button + fill_image/vector_image/slide_image，dia8，第19章）；
- 标准 BASE.DCL 颜色组（color_palette_1_7 等）直接复用。

【推断】这些 DCL 案例非常适合作为 Skill 蒸馏的**示例库**：每个范例 = DCL 布局 + 控制函数 + 数据处理三件套，可转化为 Skill 的"对话框场景模板"。

## 4. 代码风格建议（提炼）

【推断】（基于四例 + 三源码 + dia5a 案例的一致性归纳）：
1. 主命令 `defun c:xx` + 子函数拆分，子函数命名带前缀（如 dcl_、ok_、sub_）；
2. 注释使用 `;`，关键组码（40 字高、8 图层、10/11 端点）标注含义；
3. 数值与字符串转换集中处理（rtos/itoa/atof/atoi）；
4. 选择集过滤条件写在 ssget 过滤表里（`'((0 . "TEXT"))`），减少无效处理；
5. 命令函数以 `(princ)` 收尾，避免 nil 回显；
6. 涉及 command 时先 `(setvar "cmdecho" 0)`，结束恢复。
*（内容由AI生成，仅供参考）*
*（内容由AI生成，仅供参考）*
