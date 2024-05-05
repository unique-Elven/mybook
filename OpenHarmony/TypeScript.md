# 变量声明
Tyescript在JavaScript的基础上加入了静态类型检查功能，因此每一个变量都有固定的数据类型
```TS
let msg:string='hello string'
声明变量关键字 变量名 数据类型=
const代表常量
```
示例代码
```TS
//string： 字符串，可以用单引号或双引号
let msg:string = 'hello'

//number: 数值，整数，浮点数都可
let age:number = 21

//boolean
let finished: boolean = true

//any：不确定类型，可以是任意类型
let a:any = 'Elvan'
a=21
//union：联合类型，可以是多个指定类型中的一种
let u:string|number|boolean = 'Elven'

//Object：对象
let p = {name:'Elven',age:21}
console.log(p.name)
console.log(p['namne'])

//Array：数组，元素可以是任意其他类型
let names:Array<string>=['Elvan','lan']
let ages:number[]=[21,18]
console.log(names[0])
```



# 条件控制

TypeScript与大多数开发语言类似，支持基于if-else和switch的条件控制
```TS
let num:number = 21

if(num%2 === 0){
	console.log(num + '是偶数')
}else{
	console.log(num + '是奇数')
}

if(nunm>0){
	console.log(num+'是正数')
}else if(num<0){
	console.log(num+'是负数')
}else{
	console.log(num+'为0')
}
```

```TS
let grade:string = "A"
switch(grade){
	case 'A':{
		console.log('优秀')
		break
	}
	case 'B':{
		console.log('合格')
		break
	}
	case 'C':{
		console.log('不合格')
		break
	}
	default:{
		console.log('非法输入')
		break
	}
}
```


# 循环迭代
支持for和while
并为一些内置类型如Array等提供了快捷迭代语法
```TS
//普通for
for(let i = 1;i<=10;i++){
	console.log('点赞'+i+'次')
}

//while
let i = 1;
while(i <= 10){
	console.log('点赞'+i+'次')
	i++;
}
```

```TS
//定义数组
let names:string[] = ['Elven','Ro']

//for in 迭代器，遍历得到数组角标
for (const i in names){
	console.log(i + ':' + names[i])
}

//for of 迭代器，直接得到元素
for (const name of names){
	console.log(name)
}
```
# 函数
TS通常利用function关键字声明函数，并且支持可选参数，默认函数，箭头函数等特殊语法

```TS
//无返回值函数，返回值void可以省略
function sayHello(name:string):void{
	console.log('你好，'+name+'!')
}
sayHello('Elven')

//有返回值函数
function sum(x:number,y:number):number{
	return x+y
}
let result = sum(21,18)
console.log('21+18='+result)


//箭头函数
let sayHi=(name:string) =>{
	console.log('你好，' + name + '!')
}
sayHi('Elven')

//可选参数，在参数名后加？，表示该函数是可选的
function sayBye(name?:string){
	//判断name是否有值，如果无值则给一个默认值
	name = name ? name:'陌生人'
	console.log("Bye," + name + '!')
}
sayBye('Elven')
sayBye()


//参数默认，在参数后面赋值，表示参数默认值
//如果调用着没有传参，则使用默认值
function saybyby(name:string = '陌生人'){
	console.log('你好，'+name+'!')
}
saybyby('jack')
saybyby()
```

# 类和接口
TS具备面向对象编程的基本语法，例如interface、class、enum等。也具备封装、继承、多态等面向对象基本特征

```TS
//定义枚举
enum Msg{
	HI = 'Hi'
	HELLO = 'Hello'
}

//定义接口，抽象方法接受枚举类型
interface A {
	say(msg:Msg):void
}

//实现接口
class B implement A {
	say(msg:Msg):void{
		console.log(msg+',I am B')
	}
}

//初始化对象
let a:A = new B()

//调用方法，传递枚举参数
a.Say(Msg.HI)
```

```TS
//定义矩形变量
class Rectangle{
	//成员变量
	privarte width:number
	privarte length:number
	constructor(width:number,length:number){
		this.width = width
		this.length = length
	}
	//成员方法
	public area():number{
		return this.width*this.length
	}
}

//定义正方形
class Square extends Rectangle{
	constructor(side:number){
		//调用父类构造
		super(side,side)
	}
}

let s = new Square(10)
console.log('正方形面积为：'+s.area())
```

# 模块开发
