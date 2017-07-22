title: line-height
speaker: 之彦
url: https://github.com/ksky521/nodePPT
transition: cards
files: /js/demo.js,/css/demo.css

[slide]

# line-height与vertical-align
## 之彦

[slide]
# line-height指的是什么？
![](/images/line.png)

[slide]
# line-height:normal是怎么计算的？
https://codepen.io/zhiyan/pen/RgVyEd

[slide]
结论： line-height:normal根据字体的不同而变化

[slide]
# font-size指的是什么？
![](/images/font-size.jpg)

[slide]
# 概念: line-box与content-area
![](/images/content.png)

[slide]
结论：
1. line-box是line-height的区域， 多个不同line-height元素内联，line-box是叠加厚的最大区域
2. content-area大小与字体设计有关系，牵扯到字体上升边距，下降边距，大小写
3. content-area是该字体line-height设为normal时的区域

[slide]
# content-area一定小于line-height吗？
https://codepen.io/zhiyan/pen/XgRYxo

[slide]
结论： 
1. line-height:1 经常会出问题
2. 95%的字体的content-area都大于1， 从0.618到3.378不等

[slide]
# 图片的下边缝隙是怎么回事？
https://codepen.io/zhiyan/pen/dRWKLo

[slide]
# 结论
1. 图片默认是inline元素
2. 内联图片的baseline和bottom重合，都是底边
3. 图片空隙是默认vertical-align:baseline造成的，字体的baseline一般不与底边重合，字体的baseline与图片底边对齐后，下面仍需要一定的间距

[slide] vertical-align取值
* top/bottom
* middle
* text-top/text-bottom: 对齐content-area
