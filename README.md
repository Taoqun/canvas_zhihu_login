> - canvas 封装方法，增加点击，移动效果，配置颜色线条粗细功能
> - 在线查看 http://jsrun.net/3PkKp/edit
> - 传到github了，以后也许用得着，地址: https://github.com/Taoqun/canvas_zhihu_login

先上效果：

![GIF.gif](http://ocrcrbkp1.bkt.clouddn.com/GIF.gif)

点击效果：

![GIF2.gif](http://ocrcrbkp1.bkt.clouddn.com/GIF2.gif)

> - 默认生成随机大小的100个小点点
> - 鼠标移动，修改鼠标小点点的xy轴坐标
> - 鼠标点击，增加随机大小小点点
> - 增加离屏canvas 优化性能
> - 修复鼠标移出，去除鼠标效果的小点点
> - 增加暂停开始功能

index.html
```
<canvas id="root" width="1000" height="500"></canvas>
```

index.js
```
var root = document.querySelector("#root")
var a = new CanvasAnimate(root,{length:80,clicked:true,moveon:true})
    a.Run() // 开始运行
    a.pause() //  暂停
```

CanvasAnimate.js

> 说明，配置参数
- `length`  `Number`   生成的小点点数量
- `RoundColor`  `String` 小点点颜色
- `LineColor` `String` 线条颜色
- `LineWeight` `Number` 线条宽度
- `clicked` `Boolean` 鼠标点击是否增加连接小点点
- `moveon` `Boolean` 鼠标移动是否增加连接效果

-----

> 方法
- Run 开始运行
- pause 暂停动画 或者 开始动画 切换

```
function CanvasAnimate(Dom,options){
    options = options || {}
    this.Element = Dom
    this.cvs = Dom.getContext("2d")
    this.off_cvs = document.createElement("canvas")
    this.off_cvs.width = Dom.width
    this.off_cvs.height = Dom.height
    this.Dom = this.off_cvs.getContext("2d")
    this.width = Dom.width
    this.height = Dom.height
    this.length = options.length || 100
    this.RoundColor = options.RoundColor || "#999" 
    this.RoundDiameter = options.RoundDiameter || 2
    this.LineColor = options.LineColor || "#ccc"
    this.LineWeight = options.LineWeight || 1
    this.clicked = options.clicked || false
    this.moveon = options.moveon || false
    this.list = []
    this.paused = true
}
CanvasAnimate.prototype.Run = function(){
    if( this.clicked ){
        this.Element.addEventListener( "click",this.Clicked.bind(this) )
    }
    if( this.moveon ){
        this.Element.addEventListener( "mousemove",this.moveXY.bind(this) )
        this.Element.addEventListener( "mouseout",this.moveoutXY.bind(this) )
    }
    this.Draw( this.getLength() )
}
CanvasAnimate.prototype.getLength=function(){
    let arr = []
    for(let i=0;i< this.length ;i++){
        let obj = {}
            obj.x = parseInt( Math.random() * this.width )
            obj.y = parseInt( Math.random() * this.height )
            obj.r = parseInt( Math.random()*10 )
            obj.controlX = parseInt( Math.random()*10 ) > 5 ? "left":"right"
            obj.controlY = parseInt( Math.random()*10 ) > 5 ? "bottom":"top"
        arr.push(obj)
    }
    return arr
}
CanvasAnimate.prototype.Draw = function(list){
    let new_arr = []
    let line_arr = []

    list.map((item,index)=>{
        let xy = this.ControlXY(item)
        let obj = this.ControlRound(xy)
        new_arr.push( obj )
    });
    
    new_arr.map((item1,index1)=>{
        new_arr.map((item2,index2)=>{
            if(item1 !== item2){
                let x = item1.x - item2.x
                let y = item1.y - item2.y
                if( Math.abs(x)< 100 && Math.abs(y)<100 ){
                    let obj = {
                        x1:item1.x,
                        y1:item1.y,
                        x2:item2.x,
                        y2:item2.y,
                    }
                    line_arr.push(obj)
                }
            }
        })
    })

    this.drawLine(line_arr)
    
    new_arr.map((item)=>{
        this.drawRound(item)
    })

    this.list = new_arr

    this.cvs.drawImage(this.off_cvs,0,0,this.width,this.height)
    
    setTimeout(()=>{
        if(this.paused){
            this.next()
        }
    },50)
}
CanvasAnimate.prototype.next = function(){
    this.cvs.clearRect( 0,0,this.width,this.height )
    this.Dom.clearRect( 0,0,this.width,this.height )
    this.Draw( this.list )
}
CanvasAnimate.prototype.drawRound = function(obj){
    let {x,y,r} = obj
    this.Dom.beginPath()
    this.Dom.arc( x,y,r, 0, 2*Math.PI )
    this.Dom.fillStyle = this.RoundColor
    this.Dom.fill()
    this.Dom.closePath()
}
CanvasAnimate.prototype.drawLine = function(list){
    list.map( (item)=>{
        this.Dom.beginPath()
        this.Dom.moveTo( item.x1,item.y1 )
        this.Dom.lineTo( item.x2,item.y2 )
        this.Dom.lineWidth = this.LineWeight
        this.Dom.strokeStyle = this.LineColor
        this.Dom.stroke()
        this.Dom.closePath()
    })
}
CanvasAnimate.prototype.ControlXY = function(obj){
    if(obj.x >= (this.width - obj.r) ){
        obj.controlX = 'left'
    }else if( obj.x <= parseInt(obj.r/2)  ){
        obj.controlX = "right"
    }

    if( obj.y >= (this.height - obj.r) ){
        obj.controlY = "bottom"
    }else if( obj.y <= parseInt(obj.r/2) ){
        obj.controlY = "top"
    }
    return obj
}
CanvasAnimate.prototype.ControlRound = function(obj){
    switch(obj.controlX){
        case "right":
            obj.x++;
            break;
        case "left":
            obj.x--;
            break;
    }
    switch(obj.controlY){
        case "top":
            obj.y++;
            break;
        case "bottom":
            obj.y--;
            break;
    }
    return obj
}
CanvasAnimate.prototype.Clicked = function(event){
    let obj = {}
        obj.x = event.clientX
        obj.y = event.clientY
        obj.r = parseInt( Math.random()*10 )
        obj.controlX = parseInt( Math.random()*10 ) > 5 ? 'left' :'right'
        obj.controlY = parseInt( Math.random()*10 ) > 5 ? 'bottom' :'top'
    this.list.push(obj)
}
CanvasAnimate.prototype.moveXY = function(event){
    let obj = {}
        obj.x = event.clientX
        obj.y = event.clientY
        obj.r = 0
        obj.move = true
    if( this.list[0]["move"] ){
        this.list[0]["x"] = obj.x
        this.list[0]["y"] = obj.y
        this.list[0]["r"] = 1
    }else{
        this.list.unshift(obj)
    }
}
CanvasAnimate.prototype.moveoutXY = function(event){
    this.list.shift()
}
CanvasAnimate.prototype.pause = function(){
    this.paused = !this.paused
    if( this.paused){
        this.Draw(this.list)
    }
}


```
