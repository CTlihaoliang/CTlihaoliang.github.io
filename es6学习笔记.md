##ES6学习笔记
###一、let和const
let语法类似var,用来声明变量，但需注意：

- let声明的变量在所属的代码块内有效

```
for(let i = 0;i < 10;i++){
console.log(i);
}
consle.log(i);//ReferenceError
```
  
- let声明的变量不存在变量提升，即必须先声明再使用，以下使用是不安全的：

```
var tmp = 123;
	
if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```		
- 不允许重复声明(函数内部以及相同作用域内)

```
function func(a){
	let a = 10;// ReferenceError
}
	
function func(){
	let a = 10;
	var a = 1;// ReferenceError
}
```	
		
const声明一个只读的常量，一旦声明，常量的值就不能改变，也就是说必须立马初始化。

- const声明的常量也不存在变量提升，同样存在暂时性死区
- const实际保证的并不是变量的值不得改动，而是变量只想的那个内存地址不得改动

```
const a = [];
a.push('xxx');//是合法的
```
###二、变量的解构赋值
1、数组的解构赋值

 数组的解构赋值可以理解为模式匹配，即等号左边的模式尽量去匹配等号右边的模式
 
```
let [x,y,z] = [1,2,3]
let [foo,bar] = [1];
console.log(foo)//1
console.log(bar) // undefined
```
	
 		
 左侧没有被匹配的变量将会被赋值为undefined;ES6内部使用严格相等的运算符（===)来判断一个位置是否有值，所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。
 
 2、对象的解构赋值
 
 对象的解构是跟据左侧变量与右侧对象的同名属性进行匹配的。对象解构赋值的内部机制是先找到同名属性，然后再赋给对应的变量，真正赋值的是后者，而不是前者。
 
 ```
let {foo:bar} = {foo:"aaa",bar:"xxx"}
console.log(bar)//aaa	
 ```	
 3、字符串的解构赋值
 
 字符串在解构赋值时，字符串被转换了一个类似数组的对象。
 
```
const [a,b,c,d,e] = "hello";
let { length:len } = "hello";
console.log(len)// 5
``` 		

4、数值和布尔值的解构赋值

数值和布尔值在解构赋值时则会先转为对象

```
let {toString:a} = 12;
a === Number.prototype.toString // true;
	
let {toString:a} = true;
a === Boolean.prototype.toString // true;
```
		
 上面的代码中，数值和布尔值的包装对象都有toString属性，因此变量a都能取到值。
 
5、函数参数的解构赋值

函数的参数也是可以使用解构赋值的

```
function add([a,b]){
	return a+b;
}
add([1,2]);//3
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
```

###解构赋值的作用
1、交换变量的值

```
let x = 1;
let y = 2;
[x,y] = [y,x];
```
2、可以方便的将参数和变量名对应起来

```
function foo([x,y,z]){ ... }
foo([1,2,3]);
function bar({x,y,z}){ ... }
bar({x:1,y:2,z:3})
```
3、作为函数参数的默认值

```
function test(x = 1,y=2){ ... }
```

###三、字符串的扩展

1、ES6提供几种新方法：

 - includes():返回布尔值，表示是否找到了参数字符串
 - startsWith():返回布尔值，表示参数字符串是否在原字符串的头部
 - endsWith():返回布尔值，表示参数字符串是否在原字符串的尾部   
 
 这三个方法都支持第二个参数，表示开始搜索的位置。
 
 - repeat(n):返回一个新字符串，表示将原字符串重复n次
 - padStart(n,str):字符串长度自动补全，用于头部补全，n是指定字符串的最小长度，str是用来补全的字符串，如果省略第二个参数，则用空字符串补全
 - padEnd(n,str):字符串长度自动补全，用于尾部补全，n是指定字符串的最小长度，str是用来补全的字符串，如果省略第二个参数，则用空字符串补全
 
 
2、模板字符串
 
 ```
 let str = `
 	<ul>
 		<li>first</li>
 	</ul>
 `
 ```
 这样str会原模原样输出，如果想去掉空格可以调用trim()方法。模板字符串中嵌入变量，需要将变量名写在${}之中。
 
###四、数值的扩展
 
 1、ES6在Number对象上新提供几种方法:    
 
 - Number.isFinite():用来检查一个数值是否为有限的,对于非数值一律返回false
 - Number.isNaN():来检查一个值是否为NaN，只有对于NaN才返回true，非NaN一律返回false
 - Number.parseInt():跟之前的全局parseInt方法一致
 - Number.parseFloat():跟之前的全局parseFloat方法一致
 - Number.isInteger():用来判断一个值是否为整数，在JavaScript内部，整数和浮点数是同样的存储方法，所以3和3.0被视为同一个值

###五、函数的扩展

1、为函数的参数指定默认值：

```
	function bar(x = 1,y = 2){
		this.x = x;
		this.y = y;
	}

```

指定了默认值后，函数的length属性，将返回没有指定默认值的参数个数，也就是说：指定了默认值后，length属性将失真。  

一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域，例如:

```
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2) // 2
```

2、rest 参数

形式为（...变量名），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest参数搭配的变量是一个数组，改变量将多余的参数放进数组中。

```
function add(...values) {
  let sum = 0;
  for (var val of values) {
    sum += val;
  }
  return sum;
}
add(2, 5, 3) // 10
```

注意，rest 参数之后不能再有其他参数（即只能是最后一个参数），否则会报错。

3、箭头函数

```
var foo = v => v;
等价于
var foo = function(v){
	return v;
}
var sum = (num1,num2) => num1 + num2;
等价于
var sum = function(num1.num2) {
	return num1 + num2;
}
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。使用箭头函数需要注意：   
1、函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。   
2、不可以当做构造函数 。   
3、不可以使用arguments对象，该对象在函数体内不存在，如果要用，可以用rest参数代替。   
4、不可以使用yeild命令，因此箭头函数不能用作Generator函数。

###六、数组的扩展

1、扩展运算符（...）   
扩展运算符是三个点（...）。它好比rest参数的逆运算，将一个数组转为用逗号分隔的参数序列。由于扩展运算符可以展开数组，所以不再需要apply方法，将数组转为函数的参数了。例如：

```
Math.max.apply(null,[1,4,12]);
Math.max(...[1,4,12]);
Math.max(1,4,12);

```
合并数组    

```
const a = [22,33,44]    
const b = [1,...a]
```
与结构赋值结合

```
const a = list[0],rest = list.slice(1);
等价于
[a,...rest] = list;
```
将字符串转换为数组

```
[...'hello']
```

2、Array.from()    
Array.from方法用于将两类对象转为真正的数组：类似数组的对象和可遍历（iterable）的对象。常见的类似数组是将DOM操作返回的NodeList集合以及函数内部的arguments对象，Array.from都可以将它们转为正在的数组。

```
function foo(){
	var aa = Array.from(arguments);
}
```

如果参数是一个真正的数组，Array.from会返回一个一模一样的新数组。

```
Array.from([1,2,3]) //1,2,3;
```

任何有length属性的对象，都可以通过Array.from方法转为数组

```
Array.from({length:3})//[undefined,undefined,undefined]
```

Array.from还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组。

```
Array.from([1,2,3],(x) => x*x);//[1,4,9]
```

3、Array.of()    
Array.of方法用于将一组值，转换为数组

```
Array.of(1,2,3) // [1,2,3]
Array.of(2); // [2]
```

4、数组实例的find()和findIndex()    
数组实例的find方法，用于找出第一个符合条件的数组成员。他的参数是一个毁掉函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员，如果没有符合条件的成员，则返回undefined。

```
[1, 4, -5, 10].find((n) => n < 0) // -5
```

数组实例的findIndex方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。

```
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```
这两个方法都可以接受第二个参数，用来绑定回调函数的this对象。

5、数组实例的fill()   
fill方法使用给定值，填充一个数组

```
['a','b','c'].fill(4); // [4,4,4]
```
6、数组实例的entries(),keys()和values()   
keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

7、数组实例的includes()    
Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似

```
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

该方法的第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。

###七、对象的扩展

1、属性的简洁表示法     
ES6 允许直接写入变量和函数，作为对象的属性和方法，属性名为变量名， 属性值为变量的值。这样的书写更加简洁。

```
let aa = 'xxx';
const bar = {
	name:"ddd",
	aa,
	hello(){
		console.log(222);
	}
}
```

2、属性名表达式    
ES6允许字面量定义对象时，用表达式作为对象的属性名，即把表达式放在方括号内。

```
let lastWord = 'last word';
const a = {
  'first word': 'hello',
  [lastWord]: 'world'
};
a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"
```

表达式还可以用于定义方法名。

```
let hello = "foo";
let obj = {
  [hello]() {
    return 'hi';
  }
};
obj.foo() 
```

3、Object.is()   
ES5比较两个值是否相等，只有两个运算符：相等运算符和严格相等运算符。它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。Object.is()则可以用来解决这个问题

4、Object.assign()   
Object.assign方法用于对象的合并，将源对象的所有可枚举属性，复制到目标对象。Object.assign方法的第一个参数是目标对象，后面的参数都是源对象。    
Object.assign拷贝的属性是有限制的，只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（enumerable: false）。

注意：

  - Object.assign方法实行的是浅拷贝，而不是深拷贝
  - 如果源对象中的属性和目标对象的属性同名，则处理方法是替换而不是添加
  - Object.assign可以用来处理数组，但是会把数组视为对象。

  ```
  Object.assign([1, 2, 3], [4, 5])// [4, 5, 3]
  ```
  
  
目前，有四个操作会忽略enumerable为false的属性。
	
 - for...in循环：只遍历对象自身的和继承的可枚举的属性。    
 - Object.keys()：返回对象自身的所有可枚举的属性的键名。    
 - JSON.stringify()：只串行化对象自身的可枚举的属性。     
 - Object.assign()： 忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性。

 5、Object.keys()，Object.values()，Object.entries()    
 ES5 引入了Object.keys方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名。
 
 ```
 var obj = { foo: 'bar', baz: 42 };
Object.keys(obj)
// ["foo", "baz"]
 ```  
 
 Object.values方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值。
 
 ```
 const obj = { foo: 'bar', baz: 42 };
Object.values(obj)
// ["bar", 42]
 ```
 
 Object.entries方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组。
 
 ```
 const obj = { foo: 'bar', baz: 42 };
Object.entries(obj)
// [ ["foo", "bar"], ["baz", 42] ]
 ```
 
 6、对象的扩展运算符     
 ES2017 将这个运算符引入了对象。
 
 ```
 let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
 ```
 
 7、Null 传导运算符    
 编程实务中，如果读取对象内部的某个属性，往往需要判断一下该对象是否存在，现在有更方便的写法：
 
 ```
 obj?.name
 ```
 
 ###Set 和 Map 数据结构
  
  


















 
 
 
 

 
 
 		
 		

 
 

		
		

		
