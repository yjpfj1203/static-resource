markdown语法

# 一：标题
```
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```
效果如下：
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题

# 二：字体
```
**这是加粗的文字**
*这是倾斜的文字*`
***这是斜体加粗的文字***
~~这是加删除线的文字~~
```
效果如下：<br/>
**这是加粗的文字** <br/>
*这是倾斜的文字* <br/>
***这是斜体加粗的文字*** <br/>
~~这是加删除线的文字~~ <br/>

# 三：引用
```
>这是引用的内容
>>这是引用的内容
>>>>>>>>>>这是引用的内容
```
效果如下：
>这是引用的内容
>>这是引用的内容
>>>>>>>>>>这是引用的内容

# 四：分割线
三个或者三个以上的 - 或者 * 都可以。
```
---
----
***
*****
```
效果如下：
---
----
***
*****

# 五：图片
语法：
```
![图片alt](图片地址 ''图片title'')

图片alt就是显示在图片下面的文字，相当于对图片内容的解释。
图片title是图片的标题，当鼠标移到图片上时显示的内容。title可加可不加
```
示例：
```
![blockchain](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/
u=702257389,1274025419&fm=27&gp=0.jpg "区块链")
```

![blockchain](https://b-ssl.duitang.com/uploads/item/201509/30/20150930142149_wAdk5.jpeg "区块链")

# 六：超链接
语法：
```
[超链接名](超链接地址 "超链接title")
title可加可不加
```
示例：
[简书](http://jianshu.com)
[百度](http://baidu.com)

# 七：列表
## 1、无序列表
语法：无序列表用 - + * 任何一种都可以
```
- 列表内容
+ 列表内容
* 列表内容

注意：- + * 跟内容之间都要有一个空格
```
- 列表内容
+ 列表内容
* 列表内容

## 2、有序列表
语法：数字加点
```
1. 列表内容
2. 列表内容
3. 列表内容

注意：序号跟内容之间要有空格
```
1. 列表内容
2. 列表内容
3. 列表内容

## 3、列表嵌套
语法：上一级和下一级之间敲三个空格即可
+ 一级有序列表内容
   * 二级无序列表内容
   * 二级无序列表内容
   * 二级无序列表内容
+ 一级有序列表内容
   * 二级有序列表内容
   * 二级有序列表内容
   * 二级有序列表内容
+ 一级有序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
* 一级有序列表内容
   * 二级有序列表内容
   * 二级有序列表内容
   * 二级有序列表内容

# 八：表格
语法：
```
姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
```
姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
