---
title: Android Studio 的快捷键
date: 2014-02-12 01:29:16
tags: 快捷键
categories: 技术
---



#### IDE

	按键						说明
	F1							帮助
	Alt+F1					查找文件所在目录位置
	Alt+1					快速打开或隐藏工程面板
	Ctrl+Alt+S				打开设置对话框
	Alt+Home				跳转到导航栏
	Esc						光标返回编辑框
	Shift+Esc				光标返回编辑框,关闭无用的窗口
	Shift+Click				关闭标签页
	F12						把焦点从编辑器移到最近使用的工具窗口
	Ctrl+Alt+Y					同步
	Ctrl+Alt+S				打开设置对话框
	Alt+Shift+Inert			开启/关闭列选择模式
	Ctrl+Alt+Shift+S		打开当前项目/模块属性
	Alt+Shift+C				查看文件的变更历史
	Ctrl+Shift+F10			运行
	Ctrl+Shift+F9			debug运行
	Ctrl+Alt+F12				资源管理器打开文件夹


<!--more-->
#### 编辑

	按键									说明
	Alt+J								多行编辑
	Alt+MouseDrag						鼠标选中区域
	Ctrl+C								复制当前行或选中的内容
	Ctrl+D								粘贴当前行或选中的内容到下一行
	Ctrl+X								剪切当前行或选中的内容
	Ctrl+Y								删除行
	Ctrl+Z								倒退
	Ctrl+Shift+Z						向前
	Alt+Enter							自动修正
	Ctrl+Alt+L							格式化代码
	Ctrl+Alt+I							将选中的代码进行自动缩进编排
	Ctrl+Alt+O							优化导入的类和包
	Ctrl+Alt+D							对选中内容添加包围代码
	Ctrl+Alt+J							用动态模板环绕
	Alt+Insert							得到一些Intention Action，可以生成构造器、Getter、Setter、将 == 改为 equals() 等
	Ctrl+Shift+V						选最近使用的剪贴板内容并插入
	Ctrl+Alt+Shift+V					简单粘贴
	Ctrl+Shift+Insert					选最近使用的剪贴板内容并插入（同Ctrl+Shift+V）
	Ctrl+Enter							在当前行的上面插入新行，并移动光标到新行（此功能光标在行首时有效）
	Shift+Enter						在当前行的下面插入新行，并移动光标到新行
	Ctrl+J							自动代码
	Ctrl+Alt+T						把选中的代码放在 try{} 、if{} 、 else{} 里
	Shift+Alt+Insert					竖编辑模式
	Ctrl+ /								注释 //
	Ctrl+Shift+ /						注释 /…/
	Ctrl+Shift+J						合并成一行
	F2/Shift+F2						跳转到下/上一个错误语句处
	Ctrl+Shift+Back					跳转到上次编辑的地方
	Ctrl+Alt+Space					类名自动完成
	Shift+Alt+Up/Down					内容向上/下移动
	Ctrl+Shift+Up/Down					语句向上/下移动
	Ctrl+Shift+U					大小写切换
	Tab								代码标签输入完成后，按 Tab，生成代码
	Ctrl+Backspace					按单词删除
	Ctrl+Shift+Enter					语句完成

#### 文件

	按键							说明
	Ctrl+F12					显示当前文件的结构
	Ctrl+H						显示类继承结构图
	Ctrl+Q						显示注释文档
	Ctrl+P						方法参数提示
	Ctrl+U						打开当前类的父类或者实现的接口
	Alt+Left/Right				切换代码视图
	Ctrl+Alt+Left/Right			返回上次编辑的位置
	Alt+Up/Down					在方法间快速移动定位
	Ctrl+B						快速打开光标处的类或方法
	Ctrl+W						选中代码，连续按会有其他效果
	Ctrl+Shift+W					取消选择光标所在词
	Ctrl+ - / +					折叠/展开代码
	Ctrl+Shift+ - / +				折叠/展开全部代码
	Ctrl+Shift+.					折叠/展开当前花括号中的代码
	Ctrl+ ] / [					跳转到代码块结束/开始处
	F2 或 Shift+F2				高亮错误或警告快速定位
	Ctrl+Shift+C					复制路径
	Ctrl+Alt+Shift+C			复制引用，必须选择类名
	Alt+Up/Down					在方法间快速移动定位
	Shift+F1				要打开编辑器光标字符处使用的类或者方法 Java 文档的浏览器
	Ctrl+G					定位行

#### 查找

	按键							说明
	Ctrl+F					在当前窗口查找文本
	Ctrl+Shift+F			在指定环境下查找文本
	F3						向下查找关键字出现位置
	Shift+F3				向上一个关键字出现位置
	Ctrl+R					在当前窗口替换文本
	Ctrl+Shift+R			在指定窗口替换文本
	Ctrl+N					查找类
	Ctrl+Shift+N			查找文件
	Ctrl+Shift+Alt+N		查找项目中的方法或变量
	Ctrl+B					查找变量的来源
	Ctrl+Alt+B				快速打开光标处的类或方法
	Ctrl+Shift+B			跳转到类或方法实现处
	Ctrl+E					最近打开的文件
	Alt+F3					快速查找，效果和Ctrl+F相同
	F4						跳转至定义变量的位置
	Alt+F7					查询当前元素在工程中的引用
	Ctrl+F7					查询当前元素在当前文件中的引用，然后按 F3 可以选择
	Ctrl+Alt+F7				选中查询当前元素(方法或元素)在工程中的引用
	Ctrl+Alt+H				打开方法调用结构层级的窗口
	Ctrl+Shift+F7			高亮显示匹配的字符，按 Esc 高亮消失
	Ctrl+Shift+Alt+N		查找类中的方法或变量
	Ctrl+F12				查找类中的方法
	Ctrl+Shift+O			弹出显示查找内容
	Ctrl+Alt+Up/Down		快速跳转搜索结果
	Ctrl+Shift+S			高级搜索、搜索结构

#### 重构

	按键	                  说明
	F5					复制
	F6					移动
	Alt+Delete			安全删除
	Ctrl+U				转到父类
	Ctrl+O				重写父类的方法
	Ctrl+I				实现方法
	Ctrl+Alt+N			内联
	Ctrl+Alt+Shift+T	弹出重构菜单
	Shift+F6			重构-重命名
	Ctrl+Alt+M			提取代码组成方法
	Ctrl+Alt+C			将变量更改为常量
	Ctrl+Alt+V		定义变量引用当前对象或者方法的返回值
	Ctrl+Alt+F		将局部变量更改为类的成员变量
	Ctrl+Alt+P		将变量更改为方法的参数

#### 调试

	按键						说明
	F8						跳到下一步
	Shift+F8				  跳出函数、跳到下一个断点
	Alt+Shift+F8			  强制跳出函数
	F7						进入代码
	Shift+F7				  智能进入代码
	Alt+Shift+F7			  强制进入代码
	Alt+F9					运行至光标处
	Ctrl+Alt+F9				强制运行至光标处
	Ctrl+F2					停止运行
	Alt+F8					计算变量值

#### VCS

	按键					说明
	Alt+ ~ VCS			操作菜单
	Ctrl+K				提交更改
	Ctrl+T				更新项目
	Ctrl+Alt+Shift+D	  显示变化

#### Others

	按键			说明
	(Ctrl+)F11	标记书签
	Shift+F11	 显示书签

#### REF

	Android Studio Jar、so、library项目依赖


Inserted from <http://blog.hwangjr.com/2015/08/12/AndroidStudio-%E5%BF%AB%E6%8D%B7%E9%94%AE/>
