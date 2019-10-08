# Markdown 学习笔记

## 一、标题
### 1、使用#标记标题
代码示例:  
```
 # 我是一级标题
 ## 我是二级标题
```
显示效果：
># 我是一级标题
>## 我是二级标题
### 2、使用=和-标记一级和二级标题
代码示例：
```
我是一级标题
===========
我是二级标题
-----------
```
显示效果： 

>我是一级标题
>===========
>我是二级标题
>-----------
## 二、段落
在段落末尾添加两个以上的空格加换行或者用两个换行实现换行。
## 三、列表
### 1、无序列表使用星号(*)、加号(+)或是减号(-)作为列表标记，显示都是·：  
代码示例：
```
* 第一项
* 第二项

+ 第一项
+ 第二项

- 第一项
- 第二项
```
显示效果：
* 第一项
* 第二项

+ 第一项
+ 第二项

- 第一项
- 第二项

### 2、有序列表使用数字并加上 . 号来表示，如：
```
1. 第一项
2. 第二项
3. 第三项
```
显示效果：
1. 第一项
2. 第二项
3. 第三项
### 3、列表嵌套
列表嵌套只需在子列表中的选项添加四个空格即可： 
代码示例：

```
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
```
显示效果：
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
### 4、任务列表

代码示例：

```
- [x] Java
- [ ] C
- [ ] C++
- [ ] Python
```

显示效果：

- [x] Java
- [ ] C
- [ ] C++
- [ ] Python

## 三、区块
代码示例：
```
> 一级区块
>> 二级区块
```
显示效果：
> 一级区块
>
> > 二级区块

区块中可以直接使用列表，列表中使用区块需要在>前加四个空格或一个tab缩进。 
代码示例：

```
* 第一项
    > 菜鸟教程
    > 学的不仅是技术更是梦想
* 第二项
```
显示效果：
* 第一项
    > 菜鸟教程 
    > 学的不仅是技术更是梦想
* 第二项
## 四、代码
### 1、用`引入代码
当代码为一行时可用。 
代码示例：

```
`System.out.println("Hello World!")`
```
显示效果： 
`System.out.println("Hello World!")`

### 2、用多个`引入代码
当代码为多行时（要在代码块里打出```字符，要在前面加其他字符，这里前面加的是不可见字符\u200b）。
代码示例：

```
​```
if(isBoolean){
    System.out.println("Hello World!")
}
​```
```
显示效果：  

```
if(isBoolean){
    System.out.println("Hello World!")
}
```
## 五、链接
链接使用方法如下：
```
[链接名称](链接地址)

或者

<链接地址>
```
### 1、简单使用
代码示例： 
`这是一个链接 [百度](https://www.baidu.com)` 
`<https://www.baidu.com>` 

显示效果： 
这是一个链接 [百度](https://www.baidu.com) 
<https://www.baidu.com>

### 2、高级链接,链接也可以用变量来代替，文档末尾附带变量地址：  
代码示例：
```
这个链接用 1 作为网址变量 [Google][1]
这个链接用 runoob 作为网址变量 [Runoob][runoob]

[1]: http://www.google.com/
[runoob]: http://www.runoob.com/
```
显示效果：
这个链接用 1 作为网址变量 [Google][1] 
这个链接用 runoob 作为网址变量 [Runoob][runoob]

[1]: http://www.google.com/
[runoob]: http://www.runoob.com/

## 六、图片
Markdown 图片语法格式如下：
```
![alt 属性文本](图片地址)
![alt 属性文本](图片地址 "可选标题")
```
### 1、简单使用
代码示例：
```
![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png)  
![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png "RUNOOB")
```
显示效果：
![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png) 
![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png "RUNOOB")

### 2、也可以像网址那样对图片网址使用变量:
```
这个链接用 1 作为网址变量 [RUNOOB][1].
然后在文档的结尾位变量赋值（网址）

[1]: http://static.runoob.com/images/runoob-logo.png
```
显示效果：
这个链接用 1 作为网址变量 [RUNOOB][1].
然后在文档的结尾位变量赋值（网址）

[1]: http://static.runoob.com/images/runoob-logo.png

### 3、最后也可以用\<img\>标签，
代码示例：
`<img src="http://static.runoob.com/images/runoob-logo.png" width="50%">`

显示效果：
<img src="http://static.runoob.com/images/runoob-logo.png" width="50%">

## 七、表格
Markdown 制作表格使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行。

### 1、语法格式如下：
```
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |
```
显示效果：
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |

### 2、对齐方式

我们可以设置表格的对齐方式：
```
-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。
```
显示效果：
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
## 八、其他技巧
### 1、支持的 HTML 元素  
不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。 
目前支持的 HTML 元素有：\<kbd\> \<b\> \<i\> \<em\> \<sup\> \<sub\> \<br\>等 ，如： 
`使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑`

显示效果： 
使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑

### 2、转义
> Markdown 使用了很多特殊符号来表示特定的意义，如果需要显示特定的符号则需要使用转义字符，Markdown 使用反斜杠转义特殊字符：
```
**文本加粗** 
\*\* 正常显示星号 \*\*
```
显示效果： 
**文本加粗** 
\*\* 正常显示星号 \*\*

> Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：
```
\   反斜线
`   反引号
*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号
```
### 3、公式
当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。提交后，问答和文章页会根据需要加载 Mathjax 对数学公式进行渲染。如：
```
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```
显示效果：  
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$

### 4、字符格式

> 下划线：`<u>UnderLine</u>`（快捷键<kbd>ctrl</kbd>+<kbd>u</kbd>）
>
> 生成样式：<u>UnderLine</u> 

> 删除线：`~~删除文本~~`
>
> 生成样式： ~~删除文本~~

> 加粗：`**加粗**`（快捷键<kbd>ctrl</kbd>+<kbd>b</kbd>）
>
> 生成样式：**加粗**

> 倾斜：`*倾斜*`（快捷键<kbd>ctrl</kbd>+<kbd>i</kbd>）
>
> 生成样式：*倾斜*

### 5、分割线

代码示例：

`***`

显示效果：

***

### 6、注释

代码格式：

`[注释]:注释内容`

显示效果：

[注释]:注释内容

### 7、表情

代码样式：

```
:单词:
:smile:
```

显示效果：

:smile:





