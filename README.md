> canvas 封装方法，增加点击，移动效果，配置颜色线条粗细功能
> 在线查看 http://jsrun.net/3PkKp/edit
> 传到github了，以后也许用得着，地址: https://github.com/Taoqun/canvas_zhihu_login

先上效果：

![GIF.gif](http://upload-images.jianshu.io/upload_images/768057-b896a32b89842f33.gif?imageMogr2/auto-orient/strip)

点击效果：

![GIF2.gif](http://upload-images.jianshu.io/upload_images/768057-92c912b5093d6bdf.gif?imageMogr2/auto-orient/strip)

index.html
```
<canvas id="root" width="1000" height="500"></canvas>
```

index.js
```
var root = document.querySelector("#root")
var a = new CanvasAnimate(root,{length:80,clicked:true,moveon:true})
    a.Run()
```

CanvasAnimate.js

> 说明，配置参数
`length`  生成的小点点数量
`RoundColor` 小点点颜色
`LineColor` 线条颜色
`LineWeight` 线条宽度
`clicked` 鼠标点击是否增加连接小点点
`moveon` 鼠标移动是否增加连接效果

```
function CanvasAnimate(Dom,options){
    options = options || {}
    this.Element = Dom
    this.Dom = Dom.getContext("2d")
    this.width = Dom.width
    this.height = Dom.height
    this.length = options.length || 100
    this.RoundColor = options.RoundColor || "#999" 
    this.LineColor = options.LineColor || "#ccc"
    this.LineWeight = options.LineWeight || 1
    this.clicked = options.clicked || false
    this.moveon = options.moveon || false
    this.list = []
}

CanvasAnimate.prototype.Run = function(){
    if( this.clicked ){
        this.Element.addEventListener( "click",this.Clicked.bind(this) )
    }
    if( this.moveon ){
        this.Element.addEventListener( "mousemove",this.moveXY.bind(this) )
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

    setTimeout(()=>{
        this.Dom.clearRect( 0,0,this.width,this.height )
        this.Draw( new_arr )
    },50)
}
CanvasAnimate.prototype.drawRound = function(obj){
    let {x,y,r} = obj
    this.Dom.beginPath()
    let m = parseInt( Math.random() * 10)
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
    if(obj.x >= this.width ){
        obj.controlX = 'left'
    }else if( obj.x <=0  ){
        obj.controlX = "right"
    }

    if( obj.y >= this.height ){
        obj.controlY = "bottom"
    }else if( obj.y <= 0 ){
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
```