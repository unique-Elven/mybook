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