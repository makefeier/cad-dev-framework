---
AIGC: Label="1"
---

# 03 DCL 对话框开发

> 标注规则：【来源:文件名】= 书中明确说明；【推断】= 归纳延伸。
> 主要依据：《AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf》（第12~21章为 DCL 核心章节）《DCL控件属性.docx》《DCL控件属性.xlsx》《Base.dcl an Acad.dcl.docx》

---

## 1. 对话框结构与基本语法

### 1.1 DCL 是什么

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】
- DCL（Dialog Control Language）是 AutoCAD 的对话框描述语言，语法简单、编写环境不挑剔（记事本即可），通过 AutoLISP 的 DCL 函数调用激活执行；
- DCL 文件是 ASCII 文本，扩展名建议 `.dcl`；
- 一个 DCL 文件内可定义多组 `dialog`（如同一个 LSP 内可定义多个命令）；
- DCL 本身是**被动**的，所有指挥控制都由 AutoLISP 下达，连接桥梁是各对象的 **key 值**——若对象未定义 key，则无法用 AutoLISP 控制。

### 1.2 基本结构模板

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】DCL 基本结构：
```
对话框名 : dialog {
  label = "对话框标题";
  : edit_box { label = "编辑框"; key = "edit-1"; }
  : button   { label = "按钮";   key = "button-1"; }
  ok_cancel;   // 标准按钮组合
}
```
- 类比：`对话框名` 相当于 AutoLISP 的 defun 函数名；`:dialog` 相当于 defun；对象属性赋值后必须以分号 `;` 结尾（注意不是 AutoLISP 注释分号）；括号、字符串双引号必须成对；
- 对话框结束前**至少要有一个 "OK" 或 "Cancel" 按钮**；
- 对象与属性用英文小写；属性可多可少，未定义的属性自动带默认值；
- 注释方式：`//` 单行，或 `/* ... */` 多行（不能用 AutoLISP 的 `;`）。

### 1.3 语法十大守则要点

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】核心守则：
1. 文件扩展名 .dcl；2. 一个文件可多组 dialog；3. 适时注释；4. 基本结构如上；5. 大括号/双引号成对；6. 对象与属性用英文小写；7. 属性可不全定义，缺省自动带入默认值；8. 设计完先预览（VIEWDCL.LSP）；9. 加载语意有误时 AutoCAD 弹警告并写 ACAD.DCE 错误记录；10. DCL 被动，由 AutoLISP 通过 key 控制。

## 2. 常用控件与家族

### 2.1 DCL 对象四大家族

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第12章把 DCL 对象分为四类：

**家族一：基本控件（可独立定义属性）**
- `button` 按钮、`edit_box` 编辑框、`list_box` 列表框、`popup_list` 下拉列表、`radio_button` 单选按钮、`toggle` 切换开关、`slider` 滑动条、`image_button` 图像按钮、`image` 图像、`text` 文字、`text_part` 文字组成、`spacer` 空白栏、`icon_image` 图标图像（宽度默认12、比例0.66）。

**家族二：群集控件（布局容器）**
- `column` 直列、`boxed_column` 框列、`radio_column` 单选按钮列、`boxed_radio_column` 单选按钮框列、`row` 横行、`boxed_row` 框行、`radio_row` 单选按钮行、`boxed_radio_row` 单选按钮框行、`concatenation` 水平群集连结（紧凑水平拼接，常配合 text/text_part）、`paragraph` 垂直群集排列。

**家族三：BASE.DCL 整合的标准对象群**
- `errtile`（错误信息文字，宽35）、`ok_only`、`ok_cancel`、`ok_cancel_help`、`ok_cancel_err`、`spacer_0`（弹性空白）、`spacer_1`（最小空白1×1）等。

**家族四：标准默认整合 DCL**
- 直接使用 BASE.DCL 中定义好的组合（如 `ok_cancel`），加快开发。

### 2.2 控件属性总表

【来源: DCL控件属性.docx】列出 28+ 项常用属性：`action`、`alignment`、`allow_accept`、`aspect_ratio`、`big_increment`、`children_alignment`、`children_fixed_height`、`children_fixed_width`、`color`、`edit_limit`、`edit_width`、`fixed_height`、`fixed_width`、`height`、`initial_focus`、`is_bold`、`is_cancel`、`is_default`、`is_enabled`、`is_tab_stop`、`key`、`label`、`layout`、`list`、`max_value`、`min_value`、`mnemonic`、`multiple_select`、`password_char`、`small_increment`、`tabs`、`value`、`width` 等，并注明适用控件与默认值。

【来源: DCL控件属性.xlsx】以 22 类控件（dialog、button、edit_box、list_box、popup_list、radio_button、radio_column、toggle、slider、image、image_button、text、spacer 等）× 39 项属性的矩阵表说明"哪些属性可作用于哪些控件"——是查属性合法性的权威速查表。

### 2.3 关键属性说明

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】
- `key`：AutoLISP 与 DCL 沟通的唯一桥梁，必须唯一；
- `label`：控件显示文字（按钮文字、框标题等）；
- `value`：控件初始值。toggle/radio_button 只接受 `"0"/"1"`；list_box/popup_list 接受大于等于 0 的正整数字符串（表示选中项序号）；edit_box 接受任意字符串；
- `edit_width`：编辑框/下拉框输入区宽度；
- `width` / `height`：控件尺寸；`aspect_ratio` 宽高比（image 常用 0.66）；
- `list`：popup_list/list_box 预置列表，多项用 `\n` 分隔（如 `list="1:1\n1:10\n1:20"`）；
- `alignment`：对齐方式（如 centered）。

【来源: Base.dcl an Acad.dcl.docx】AutoCAD 官方 BASE.DCL / ACAD.DCL 源码定义了上述标准对象原型（dialog/cluster/tile/text/button 等），可观摩学习组合方式，但**严禁修改**（铁道版基础篇第13章明确警告初学者"千万不要轻举妄动"）。

### 2.4 第17~21章控件专项设计技巧（DCL 新手上路系列）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第三篇第16~21章为"DCL 新手上路"系列，按控件专项展开完整可运行范例（第16章编辑框/按钮/文字已见 §4 dia5a，此处提炼第17~21章）：

**第17章 切换开关（toggle）与单选按钮（radio_button）**
- dia6a：五组系统变量预设，`set_tile` 用 `(itoa (getvar ...))` 预填，确定回写用 `(setvar ... (atoi (get_tile ...)))`；取消键返回 -1；
- dia6b：属性图块插入工具，`radio_column` 内单选按钮用 `(get_attr key "label")` 取选项名，动作表达式用 `$key` 动态判断所选；配套 `while` 循环实现连续插入（不退出对话框）；
- 要点：单选按钮组用 `radio_column`/`radio_row` 保证互斥；toggle 的 value 只接受 "0"/"1"。

**第18章 列表框（list_box）与下拉列表（popup_list）**
- dia7a：词库管理，`start_list → (mapcar 'add_list 词库) → end_list` 三步填充；`$value` 返回选中序号字符串，需 `(atoi ...)` 后 `(nth ...)` 取词；点选后 `set_tile` 回填编辑框；
- dia7b（专家改进）：共享 `show_list` 子程序封装列表填充；`chk_style` 自动建立所需图层/文字样式；`strcat` 累加词库、`strcase` 统一大小写；`fixed_width_font = true` 让列表框等宽显示；
- dia7c：下拉选单（popup_list），选中项联动编辑框（`sub_pop` 用 `$value` 更新）；要点：list_box 可多行显示、popup_list 单行折叠，按选项数量取舍。

**第19章 图像（image）与图像按钮（image_button）**
- dia8a：`start_image → (fill_image 0 0 px py -2) 清底 → slide_image 显示 SLD 幻灯片 → end_image`；框尺寸用 `dimx_tile`/`dimy_tile` 取得；
- dia8b：四个图像按钮（image_button），`allow_accept = true` 支持双击直接确认；`(mapcar 'show_sld key_list sld_list)` 批量初始化；
- dia8c（专家改进）：图像区上方加 `text` 标题说明、`popup_list` 切换不同图像组、`mode_tile` 模式 2 预设焦点高亮。

**第20章 滑动条（slider）**
- dia9a：水平滑动条，属性 `min_value`/`max_value`/`small_increment`/`big_increment`；`$value` 为整数字符串，用 `(chr (+ 64 序号))` 转字母做图块名；
- dia9b：垂直滑动条，`layout = vertical`，`width = 1`、`height = 10`；
- dia9c（专家级）：滑动条 + 图像实时联动预览（`osmode` 保存/恢复、`vector_image` 参数化实时绘图、STR/DIM 图层管理、`command "dim"` 标注），是图像类控件联动的完整样板。

**第21章 结训测验（dia10a~dia10p）**
- 14 个新手常犯错误：括号不配对、字符串缺引号、dialogname 打错、`set_tile` 传非字符串、空函数、对话框激活期调用被禁函数、`command "pline"` 未闭合、`action_tile` 参数过多、`new_dialog` 参数过少、表达式多余引号、漏 DCL 控制函数导致当掉等；
- 【推断】可作 Skill 蒸馏的"自检用例库"：每个错误=一个负例，用于校验生成代码与文档示例。

## 3. AutoLISP 控制 DCL 的关键函数（第15章核心）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】完整控制函数链：

### 3.1 加载与释放
- `(load_dialog "文件")`：加载 DCL 文件，成功返回正整数值 dcl_id；失败返回 -1；dcl_id 是后续 new_dialog/unload_dialog 必需的"代码值"；
- `(unload_dialog dcl_id)`：释放内存中的 DCL 文件。

### 3.2 激活与终止
- `(new_dialog "对话框名" dcl_id [默认动作] [屏幕坐标])`：激活对话框。一个 DCL 文件可定义多个 dialog 名；坐标缺省 (-1 -1) 表示屏幕中央；若指定坐标而省略默认动作，也必须写空字符串 `""`；
- `(start_dialog)`：把控制权正式交给 DCL。**必须在 new_dialog 之后、且在此之前完成所有 set_tile/mode_tile/action_tile**；漏写 start_dialog 会导致对话框失控；
- `(done_dialog 结束码)`：终止当前对话框。结束码必须为正整数，返回给 start_dialog。**OK 按钮 key="accept"，默认结束码=1；Cancel 按钮 key="cancel"，默认结束码=0**；
- 常用写法：`(setq dd (start_dialog))`，再按 dd 值（1/2/...）用 cond 分发执行不同子程序；
- `(done_dialog n)` 会返回当前对话框位置的 2D 坐标，可存起来下次 new_dialog 定位用（如 `(setq dcl_pt (done_dialog 2))`）；
- `(term_dialog)`：终止显示当前**所有**对话框（多重/嵌套），返回 -1 给 start_dialog。

### 3.3 触发与动作
- `(action_tile "key" "动作表达式或子程序")`：触发对象时执行指定动作，**必须在 new_dialog 之后、start_dialog 之前定义**；
- 触发时自动生成六个系统变量：
  - `$key` 被触发对象的 key；`$value` 对象当前值（字符串；toggle 为 "0"/"1"，list/popup 为选中序号从 "0" 起，edit_box 为框内字符串）；
  - `$x` / `$y` 触发点的坐标（主要用于 image_button）；
  - `$data` 应用程序特定数据（需 client_data_tile 设定）；
  - `$reason` 动作发生原因（观察 edit_box/list_box/image_button/slider 用）；
- 动作表达式必须**整体放在一个字符串**内，多个动作拼接：`(action_tile "key1" "(sub_a)(sub_b)(sub_c)")`（错误：`"(sub_a)" "(sub_b)"` 多参数）；
- 字符串变量内嵌需转义：`(action_tile "key1" "(setq str1 \"ABC\")")`；带自变量子程序：`(action_tile "key1" "(sub_data \"A\" \"B\")")`。

### 3.4 数据设定与取得
- `(mode_tile "key" 模式)`：设定显示模式。模式 0=恢复作用、1=禁用（淡化、不可选）、2=设为焦点（光标默认位置）、3=选择编辑框内容（少用）、4=控制 image_button 高亮显示开关。**注意模式 0/1 与日常直觉（1开/0关）相反**；模式 4 只作用于 image_button；禁用状态下仍可用 get_tile 取值；
- `(set_tile "key" "字符串")`：设定对象值，**必须字符串**（数字用 itoa/rtos 转换）；toggle/radio 只收 "0"/"1"，list 类收序号字符串；可用 `(mapcar 'set_tile key_list val_list)` 批量设定；
- `(get_tile "key")`：取得对象值（字符串），配合 mapcar 批量获取：`(setq val_list (mapcar 'get_tile key_list))`；常用 atoi/atof 转数值后 setvar；
- `(get_attr "key" "属性名")`：取得对象指定属性的设定值（字符串）。**key 与属性名大小写敏感**：`(get_attr "kword" "edit_width")` 正确，`"EDIT_WIDTH"` 返回 ""。

### 3.5 列表框/下拉列表处理
- `(start_list "key" [操作] [index])`：开始处理。操作 1=改 index 项数据、2=加入新项、3=删除旧项并重建（默认 3，index 默认 0）；
- `(add_list "字符串")`：加入一项（与 mapcar 搭配批量加入）；
- `(end_list)`：结束处理。
- 例：`(start_list "klist1") (mapcar 'add_list data_list) (end_list)`。

### 3.6 图像/图像按钮处理
- `(start_image "key")` / `(end_image)`：图像绘制区间；
- `(dimx_tile "key")` / `(dimy_tile "key")`：取图像框右下角坐标（左上角默认 0,0）；
- `(fill_image x1 y1 w h color)`：填充色块（w/h 必须>0）；
- `(vector_image x1 y1 x2 y2 color)`：绘制向量直线；
- `(slide_image x1 y1 w h "sld名")`：显示 SLD 幻灯片；
- 特殊颜色码：-2=当前图形屏幕背景色、-15=对话框背景色、-16=对话框前景色、-18=对话框线条色；
- 惯例：`(start_image "key")` 后先 `(fill_image 0 0 px py -2)` 清空图像框，避免多次调用重叠；
- 封装共享子程序模式：`(defun show_sld (key sld) ...)` 后用 `(show_sld "sld_poly1" "exp8_1")` 调用。

## 4. 控件事件处理与数据交互流程（整合模式）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第16章给出标准交互流程（dia5a 完整案例：预设 PLINEWID/FILLETRAD/TEXTSIZE/SAVETIME 四个系统变量）：

```
(defun c:dia5a ()                 ; 主命令
  (setvar "cmdecho" 0)
  (dcl_dia5a)                     ; 调用对话框子程序
  (princ))

(defun dcl_dia5a ()               ; 对话框控制
  (setq dcl_id (load_dialog "dia5a"))
  (new_dialog "dia5a" dcl_id)
  ; 预设值：getvar 取值 → rtos/itoa 转字符串 → set_tile
  (set_tile "edit_p" (rtos (getvar "plinewid")))
  (set_tile "edit_s" (itoa (getvar "savetime")))
  ; 确定按钮动作：取数据 → 转数值 → setvar
  (action_tile "accept" "(ok_dia5a)(done_dialog 1)")
  (start_dialog)
  (unload_dialog dcl_id))

(defun ok_dia5a ()                ; 数据回写
  (setvar "plinewid" (atof (get_tile "edit_p")))
  (setvar "savetime" (atoi (get_tile "edit_s"))))
```

关键时序：`load_dialog → new_dialog → (set_tile/mode_tile/action_tile…) → start_dialog → (用户操作触发 action_tile → done_dialog 返回结束码) → 按结束码分发子程序 → unload_dialog`。

## 5. 重要禁忌（DCL 场景）

【来源: AUTOLISP@DCL基础篇(全)铁道出版社(推荐).pdf】第15章明确列出 **DCL 对话框激活期间不能调用的 AutoLISP 函数**（否则终止所有对话框并报"AutoCAD 拒绝的函数"）：`command`、`entdel`、`entmake`、`entmod`、`entget` 关联的选取类、`entsel`、`entupd`、`getangle`、`getcorner`、`getdist`、`getint`、`getkword`、`getorient`、`getpoint`、`getreal`、`getstring`、`grclear`、`grdraw`、`grread`、`grtext`、`grvecs`、`menucmd`、`nentsel`、`osnap`、`prompt`、`redraw`、`ssget`（交互式）、`textpage`、`textscr` 等。需要在对话框激活期间做的图形操作，应等 done_dialog 关闭后再执行。

【推断】因此，交互输入类需求（取点、选对象、输数字）尽量在对话框内用 DCL 控件完成，图形操作放到对话框结束后；这决定 Skill 中"对话框+命令"的拆分结构。
*（内容由AI生成，仅供参考）*
*（内容由AI生成，仅供参考）*
