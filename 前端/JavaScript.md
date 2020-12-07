# 1. 类型 值 变量

> JavaScript的类型有两种: 原始类型和对象类型

> 原始类型: 数字, 字符串, 布尔值.

> JavaScript中除去数字, 字符串, 布尔值, null, undefined 之外就是对象.
>
> 对象: JavaScript对象是属性(property)的集合
>
> 属性: 每个属性都有 "键值对" 构成. 值可以是原始类型, 也可以是对象类型.

> 函数是具有名称和参数列表的JavaScript代码片段.

> 直接量: 在程序中直接使用的数据值就是直接量.
>
> ```JavaScript
> // 以下列出的都是直接量.
> 12
> 1.2
> "hello world"
> true
> null
> ```

## 1.1 数字

> JavaScript不区分整数值和浮点数值. 
>
> JavaScript中的所有数字均采用浮点数表示.

```javascript
// JavaScript中的整数和浮点数直接量
0
3
100

3.14
23.45
.333
```



> 除了简单的运算外, 可通过 Math 来完成复杂运算

```JavaScript
Math.pow(2, 3) 	// => 8: 2的3次方.
Math.pow(16, .5)	// => 16的平方根
Math.sqrt(16)			// => 16的平方根
Math.pow(16, 1/3)	// => 16的立方根
Math.round(.6) 	// => 1.0: 四舍五入
Math.ceil(.6)		// => 1: 向上求整
Math.floor(.6)	// => 0: 向下求整
Math.abs(-5)		// => 5: 求绝对值
Math.max(x, y, z)	// => 返回最大值
Math.min(x, y, z)	// => 返回最小值
Math.random()		// => 生成一个随机数x, 0 <= x < 1.0
Math.PI					// => 圆周率
```



> JavaScript定义了全局变量 Infinity 和 NaN, 分别用来表示 正无穷大 和 非数字值. 它们都是只读的.



- JavaScript的算术运算在溢出(overflow), 下溢(underflow)或者被0整除时, 不会报错.

- 数字结果超出了JavaScript所能表示数字上限, 也就是overflow时, 结果是一个无穷大的值 => Infinity.

- 负数值超过JavaScript所能表示附属范围, 结果为负无穷大 => -Infinity.

- underflow是当运算结果无线接近于0, 并比JavaScript所能表示的最小值还小的时候发生的一种情形. 此时JavaScript返回0.
- 被0整除, JavaScript不会报错. 只是简单的返回Infinity或者 -Infinity.
- 0/0是一个意外,会返回一个非数字值 => NaN. 

> NaN比较特殊. 表示非数字值. 它和任何值都不相等, 包括它自身.
>
> 所以, 想要判断一个变量是否为NaN, 需要使用 x!=x, 这是因为当且仅当x 为NaN的时候, 它才不会等于本身, 结果才会为true. 
>
> 类似的方法有  isNaN(), 当参数为NaN或者一个非数字值(比如字符串, 对象)的时候, 返回true.
>
> isFinite(x): 当x不是NaN, Infinity, -Infinity的时候, 返回true