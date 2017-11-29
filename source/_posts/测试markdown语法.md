---
title: 测试markdown语法
date: 2017-07-18 15:58:46
tags:
- markdown
categories:
- markdown
copyright: true
---

**测试markdown语法，这边数学表达式还不能够显示**


<!--more-->

```javascript
function listCacheDelete(key) {
  var data = this.__data__,
    index = assocIndexOf(data, key);

  if (index < 0) {
    return false;
  }
  var lastIndex = data.length - 1;
  if (index == lastIndex) {
    data.pop();
  } else {
    splice.call(data, index, 1);
  }
  return true;
}

```
<i style="background-color:black;border-radius:3px;color:white;">站点配置文件</i>

```
/* global hexo */

var merge = require('./merge');

/**
 * Merge configs in _data/next.yml into hexo.theme.config.
 * Note: configs in _data/next.yml will override configs in hexo.theme.config.
 */
hexo.on('generateBefore', function () {
  if (hexo.locals.get) {
    var data = hexo.locals.get('data');
    data && data.next && merge(hexo.theme.config, data.next);
  }
});
```
{% blockquote %}
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque hendrerit lacus ut purus iaculis feugiat. Sed nec tempor elit, quis aliquam neque. Curabitur sed diam eget dolor fermentum semper at eu lorem.
{% endblockquote %}

![美丽花儿](http://ww2.sinaimg.cn/large/56d258bdjw1eugeubg8ujj21kw16odn6.jpg "美丽花儿")

![](/images/avatar.jpg)


+  轻轻的我走了， 正如我轻轻的来； 我轻轻的招手， 作别西天的云彩。
那河畔的金柳， 是夕阳中的新娘； 波光里的艳影， 在我的心头荡漾。 
软泥上的青荇， 油油的在水底招摇； 在康河的柔波里， 我甘心做一条水草！ 
* 那榆荫下的一潭， 不是清泉， 是天上虹； 揉碎在浮藻间， 沉淀着彩虹似的梦。 
寻梦？撑一支长篙， 向青草更青处漫溯； 满载一船星辉， 在星辉斑斓里放歌。 
但我不能放歌， 悄悄是别离的笙箫； 夏虫也为我沉默， 沉默是今晚的康桥！ 
悄悄的我走了， 正如我悄悄的来； 我挥一挥衣袖， 不带走一片云彩。
-  轻轻的我走了， 正如我轻轻的来； 我轻轻的招手， 作别西天的云彩。
那河畔的金柳， 是夕阳中的新娘； 波光里的艳影， 在我的心头荡漾。 
软泥上的青荇， 油油的在水底招摇； 在康河的柔波里， 我甘心做一条水草！ 

        java
javajavajava
java
java


1986. What a great season.
1986. What a great season.
1986\. What a great season.

>轻轻的我走了， 正如我轻轻的来
轻轻的我走了， 正如我轻轻的来

轻轻的我走了， 正如我轻轻的来

>>> 请问 Markdwon 怎么用？ - 小白

>> 自己看教程！ - 愤青

> 教程在哪？ - 小白


> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
> 
> 给出一些例子代码：
> 
>     return shell_exec("echo $input | $markdown_script");

<http://www.baidu.com>

# 1
## 1.1
## 1.2
### 1.2.1
# 2
# 3
# 3.1
## 3.1.1
# 4

质能守恒方程可以用一个很简洁的方程式 $E=mc^2$ 来表达。

$$\sum_{i=1}^n a_i=0$$
$$f(x_1,x_x,\ldots,x_n) = x_1^2 + x_2^2 + \cdots + x_n^2 $$
$$\sum^{j-1}_{k=0}{\widehat{\gamma}_{kj} z_k}$$


使用 Markdown[^1]可以效率的书写文档, 直接转换成 HTML[^2], 你可以使用 Leanote[^Le] 编辑器进行书写。

[^1]:Markdown是一种纯文本标记语言


[^2]:HyperText Markup Language 超文本标记语言


[^Le]:开源笔记平台，支持Markdown和笔记直接发为博文


flow
st=>start: Start:>https://www.zybuluo.com
io=>inputoutput: verification
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end
st->io->op->cond
cond(yes)->e
cond(no)->sub->io



产品|价格
-|-:
Leanote 高级账号|60元/年
Leanote 超级账号|120元/年

|学号|姓名|分数|
|-|-|-|
|小明|男|75|
|小红|女|79|
|小陆|男|92|


学号|姓名|分数
-|-|-
小明|男|75
小红|女|79
小陆|男|92


***

---

    <table>
        <tr><td>1</td><td>2</td><td>3</td><td>4</td></tr>
    </table>

<table>
    <tr><td>1</td><td>2</td><td>3</td><td>4</td></tr>
</table>