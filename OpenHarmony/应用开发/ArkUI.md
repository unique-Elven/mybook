# 组件

## Image：显示组件
![[Pasted image 20240505165718.png]]

## Text：文本显示组件
![[Pasted image 20240505172010.png]]

## TextInput：文本输入框
![[Pasted image 20240505173630.png]]

## Button：按钮组件
![[Pasted image 20240505175620.png]]

## Slider：滑动条组件

![[Pasted image 20240505180001.png]]

# 页面布局
![[Pasted image 20240505181454.png]]

![[Pasted image 20240505181800.png]]

![[Pasted image 20240505181901.png]]

![[Pasted image 20240505183705.png]]

# Ark组件-循环控制
![[Pasted image 20240506212348.png]]

![[Pasted image 20240506212435.png]]

# Ark组件-List实现滚动

![[Pasted image 20240506213017.png]]

# ArkUI-自定义组件

全局自定义构建函数和全局公共样式都需要function关键字，但是写在局部就不用了。

全局公共样式里面只能写公共通用的属性，除非使用@Extend()关键字继承，而且只能写在全局
![[Pasted image 20240507111537.png]]

![[Pasted image 20240507111707.png]]
还可以写事件方法

# ArkUI-状态管理@State装饰器
类中嵌套的类的成员对象不会重新渲染，数组内嵌套的类的成员对象也不会重新渲染但可以增加和删除元素
![[Pasted image 20240507113531.png]]

# ArkUI-状态管理-任务统计案例
```TS
//任务类  
class Task{  
  static id:number = 1  
  //任务名称  
  name:string = `任务${Task.id++}`  
  //任务状态：是否完成  
  finished: boolean = false  
}  
  
//统一的卡片样式  
@Styles function card(){  
  .width('95%')  
  .padding(20)  
  .backgroundColor(Color.White)  
  .borderRadius(15)  
  .shadow({radius:6,color:'#1f000000',offsetX:2,offsetY:4})  
}  
  
//任务完成样式  
@Extend(Text) function finishedTask(){  
  .decoration({type:TextDecorationType.LineThrough})  
  .fontColor('#B1B2B1')  
}  
  
@Entry  
@Component  
struct PropPage{  
  //任务数量  
  @State totalTask:number = 0  
  //已完成任务数量  
  @State finishTask:number = 0  
  //任务数组  
  @State tasks: Task[] = []  
  
  handlerTaskChange(){  
    //更新让我任务总数  
    this.totalTask = this.tasks.length  
    //更新已完成任务数量  
    this.finishTask = this.tasks.filter(item => item.finished).length  
  }  
  
  build(){  
    Column({space:10}){  
      //1、任务进度卡片  
      Row(){  
        Text('任务进度：')  
          .fontSize(30)  
          .fontWeight(FontWeight.Bold)  
        Stack(){  
          Progress({  
            value: this.finishTask,  
            total:this.totalTask,  
            type:ProgressType.Ring  
          })  
            .width(100)  
          Row(){  
            Text(this.finishTask.toString())  
              .fontSize(24)  
              .fontColor('#36D')  
            Text('/'+this.totalTask.toString())  
              .fontSize(24)  
          }  
        }      }      .card()  
      .margin({top:20,bottom:10})  
      .justifyContent(FlexAlign.SpaceEvenly)  
      //2、新增任务按钮  
      Button('新增任务')  
        .width(200)  
        .onClick(()=>{  
          //新增任务数据  
          this.tasks.push(new Task())  
          //更新让我任务总数  
          this.handlerTaskChange()  
        })  
      //3、任务列表  
      List({space:10}){  
        ForEach(  
          this.tasks,  
          (item:Task,index)=>{  
            ListItem(){  
              Row(){  
                Text(item.name)  
                  .fontSize(20)  
                Checkbox()  
                  .select(item.finished)  
                  .onChange(val=>{  
                    //新增任务数据  
                    item.finished = val  
                    //更新已完成任务数量  
                    this.handlerTaskChange()  
                  })  
              }  
              .card()  
              .justifyContent(FlexAlign.SpaceBetween)  
            }  
            .swipeAction({end:this.DeleteButtom(index)})  
          }  
        )  
      }  
      .width('100%')  
      .layoutWeight(1)  
      .alignListItem(ListItemAlign.Center)  
    }  
    .width('100%')  
    .height('100%')  
    .backgroundColor('#F1F2F3')  
  }  
  @Builder DeleteButtom(index:number){  
    Button(){  
      Image($r('app.media.sc'))  
        .fillColor(Color.White)  
        .width(20)  
    }  
    .width(40)  
    .height(40)  
    .type(ButtonType.Circle)  
    .backgroundColor(Color.Red)  
    .margin(5)  
    .onClick(()=>{  
      this.tasks.splice(index,1)  
      this.handlerTaskChange()  
    })  
  }  
}
```

# 状态管理-@Prop@Link@Provide@Comsume

![[Pasted image 20240507142920.png]]

# 状态管理-@Observed@ObjectLink