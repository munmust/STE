盒模型
标准盒模型和怪异盒模型
- `content-box`：默认属性。width 只包含 content
- `border-box`：width 包含 (content、padding、border)

flex
 `flex-flow`:`flex-direction` 和 `flex-wrap`
 `flex`:`flex-grow`，`flex-shrink`，`flex-basis`
 `flex-basis` 定义了该元素的**空间大小**,flex 容器里除了元素所占的空间以外的富余空间就是**可用空间**;如果没有给元素设定尺寸，`flex-basis` 的值采用元素内容的尺寸。这就解释了：我们给只要给 Flex 元素的父元素声明 `display: flex`，所有子元素就会排成一行，且自动分配小大以充分展示元素的内容
`flex-grow` 若被赋值为一个正整数，flex 元素会以 `flex-basis` 为基础，沿主轴方向增长尺寸
`flex-shrink`属性是处理 flex 元素收缩的问题。如果我们的容器中没有足够排列 flex 元素的空间，那么可以把 flex 元素`flex-shrink`属性设置为正整数来缩小它所占空间到`flex-basis`以下

默认值：flex:0 1 auto，即在有剩余空间时，只放大不缩小
flex: none，即有剩余空间时，不放大也不缩小，最终尺寸通常表现为最大内容宽度。
flex: 0，即当有剩余空间时，项目宽度为其内容的宽度，最终尺寸表现为最小内容宽度
flex: auto，即元素尺寸可以弹性增大，也可以弹性变小，具有十足的弹性，但在尺寸不足时会优先最大化内容尺寸
flex: 1，即元素尺寸可以弹性增大，也可以弹性变小，具有十足的弹性，但是在尺寸不足时会优先最小化内容尺寸

#  css variable
css变量减少样式重复定义，比如同一个颜色值要在多个地方重复使用，以前通过less和sass预处理做到，现在css变量也可以做到，方便维护，提高可读性
``` css
.box { 
	--base-size: 10; 
	width: calc(var(--base-size)* 10px);
	height: clac(var(--base-size)* 5px); 
	padding:calc(var(--base-size) * 1px);
}
```
``` javascript
// 设置变量
document.getElementById("box").style.setPropertyValue('--color', 'pink')
// 读取变量
doucment.getElementById('box').style.getPropertyValue('--color').trim() //pink
// 删除变量
document.getElementById('box').style.removeProperty('--color')
```


display：inline、block、inline-block
inline： 會依照元素中的內容來決定內容的寬高，不會佔滿容器橫向的剩餘空間，在不超過容器的狀況下，可以與其他元素並列
block：在不更改 display 的狀態下，會在容器中呈現滿版，並且可以透過 width、height、max-width、max-height 等 CSS 屬性來更改該元素的寬高，帶有 display:block 屬性的元素即便更改了寬度，剩餘的空間依然還是會在網頁上佔有空間，後續的元素並不會因為寬度縮小了，而向上並排


# BFC
块级格式化上下文，**隔离了的独立容器容器内的样式不会影响到外部的元素**
同一个BFC下外边距会发生折叠，最大的为准
高度计算会加上float的高度
区域不会和float重叠

利用它来防止margin塌陷
清除内部的浮动

绝对定位
overflow为visible之外的属性
浮动
body根元素
