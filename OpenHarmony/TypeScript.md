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
//无返回值函数，fan'b'n
```