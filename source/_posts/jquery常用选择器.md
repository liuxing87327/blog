title: "jquery常用选择器"
date: 2015-02-01 00:00:00
category: 前端
tags: jquery

---

记录一下

##选择器
###基本选择器
 `#`id —— 根据指定的id匹配一个元素
 element —— 根据“元素名”匹配元素
 .class —— 根据指定的“类名”匹配元素
 selector1,selector2,selector3... —— 组合选择器，为并集关系
 * —— 通用选择器， 匹配所有元素

###层次选择器
 $('ancestor descendant') —— 匹配ancestor下的所有descendant元素， 包括子孙节点
 $('parent > child') —— 匹配parent下的所有child元素， 注意child为parent的直接子节点， 即不包含子孙节点
 $('prev + next') —— 匹配pre元素后的next元素
 $('prev~siblings') —— 匹配prev元素后的所有siblings元素

###基本过滤选择器（针对一个集合的操作，可以把这个集合想象成ArrayList）
 :first —— 选择第一个元素， 如$('div:first')， 选择第一个div元素
 :last —— 选择最后一个元素
 :not(selector) —— 过滤掉所有与给定selector匹配的元素，如$('input:not(.myClass)')选取class不是myClass的input元素
 :even —— 选取索引是偶数的所有元素， 索引从0开始
 :odd —— 选取索引是奇数的所有元素
 :eq(index)  —— 选取索引是index的元素
 :gt(index) —— 选取索引大于index的所有元素
 :header —— 选取所有的标题元素，如h1,h2,h3等
 :animated —— 选取当前正在进行动画的的所有元素

###子元素过滤器（父元素的第几个孩子，故在使用时都会以父元素作为上下文）
 :nth-child —— 选取父元素的第几个孩子， 注意索引下标从1开始， 而:eq(index)是从0开始的
 :nth-child(even) 选取父元素下索引值是偶数的子元素
 :nth-child(odd) 选取父元素下索引值是奇数的子元素
 :nth-child(index) 选取父元素下索引值为index的子元素
 :nth-child(3n) 选取父元素下索引值是3的倍数的元素（n从0开始）
 :nth-child(3n+1) 选取父元素下索引值是3n+1的元素 （n从0开始）
 :first-child —— 父元素下的第一个孩子
 :last-child —— 父元素下的最后一个孩子
 :only-child —— 父元素下的唯一孩子（选中独生子）

###内容过滤选择器
 :contains(text) —— 选取含有文本内容为"text"的元素
 :empty —— 选取不包含子元素或文本的空元素
 :has(selector) —— 选取含有选择器所匹配的元素的元素，如$('div:has(p)')选取含有p元素的所有div元素
 :parent —— 选取含有子元素或文本的元素，如$('div:parent')选取拥有子元素或文本元素的所有div元素

###可见性过滤选择器
 :hidden ——  选取所有不可见元素，包括hidden， display:none，visibility:hidden元素，若只想选择input，则可用input:hidden
 :visible —— 选取所有可见元素

###属性过滤选择器
 [attribute] —— 选取拥有此属性的元素
 [attribute=value] —— 选取属性值为value的元素
 [attribute!=value] —— 选取属性值不等于value的元素
 [attribute^=value] —— 选取属性值以value开头的元素
 [attribute$=value] —— 选取属性值以vlue结尾的元素
 [attribute*=value] —— 选取属性值包含value的元素
 [selector1][selector2][selector3] —— 复合属性选择器，交集关系， 如$("div[id][title$='test']")选取拥有属性id，并且属性title以"test"结尾的div元素

###表单对象属性过滤器
 :enabled —— 选取所有可用元素
 :disabled —— 选取所有不可用元素
 :checked —— 选取所有被选中的元素（单选和复选框）
 :selected —— 选取所有被选中的select元素

###表单选择器
 :input —— 选取所有的input,textarea,select,button元素
 :text —— 选取所有的单行文本
 :password —— 选取所有的密码框
 :radio —— 选取所有的单选按钮
 :checkbox —— 选取所有的复选框
 :submit —— 选取所有的提交按钮
 :image —— 选取所有的图像按钮
 :reset —— 选取所有的重置按钮
 :button —— 选取所有的按钮
 :file —— 选取所有的上传文件域
 :hidden —— 选取所拥有不可见元素
 
 ##更多
 更多语法参阅API文档
 
 http://jquery.cuishifeng.cn