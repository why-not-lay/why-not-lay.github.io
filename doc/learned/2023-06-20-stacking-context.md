# 层叠上下文 (stacking context)

> 创建时间: 2023-06-20  
> 更新时间: 2023-06-20

层叠上下文是 HTML 元素在 Z 轴上位序信息的集合，类似于积木在某个方向上的堆叠情况。

层叠上下文被依据一定的[规则](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context#%E5%B1%82%E5%8F%A0%E4%B8%8A%E4%B8%8B%E6%96%87)创建，而在层叠上下文中，内部满足创建规则的子元素同样会创建一个新的层叠上下文，并且子层叠上下文是独立于父层叠上下文。

在任意一个层叠上下文里，以一定的层叠顺序赋予内部的**后代元素**特定的层级，元素的层级影响元素的先后绘制顺序。

## 参考

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context#%E5%B1%82%E5%8F%A0%E4%B8%8A%E4%B8%8B%E6%96%87)

[深入理解CSS中的层叠上下文和层叠顺序](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)
