# 一个简单的无缝滚动插件
vue2和vue3~~应该~~都可以用

## 例子：

* template
 ```
<SimpleSeamlessScrolling :list="list" >
  <template #default="{ item }">
    <div class="list-item" >{{ item }}</div>
  </template>
</SimpleSeamlessScrolling>
 ```
* script

```
import SimpleSeamlessScrolling from "simple-seamless-scrolling";
```

```
export default {
  data(){
    return {
      list: [ "吃葡萄就吐葡萄皮，不吃葡萄怎么能吐葡萄皮", "扁担长，板凳宽，后面的词我忘了", "黑发肥发灰会非法，灰化肥飞花会发黑" ]
    }
  }
}
```
 
* style

```
.list-item{
  margin-left: 10px;
}
```

## API

> **props**
>---
>| 参数 | 说明 | 类型 | 默认值 |
>|-----|:------|------|-------|
>| list | 用来渲染滚动的列表，必传，数组内数据格式不固定，一般是字符串数组，但如果没有插槽的话显示的内容可能会和你想象的不一样 | Array | - |
>| step | 滚动速度，绝对值越大越快，可以为负数 | Number | 1 | 
>| unlimited | 是否是无限滚动模式，朝一个方向无限滚动，不会再回到列表开头，需要动态向list中添加数据 | Boolean | false | 
>| direction | 方向，horizontal: 水平, vertical: 垂直 | String | horizontal | 
>| touchMove | 是否跟随手指触摸移动 | Boolean | true | 

> **event**
>---
>| 事件名 | 说明 | 回调参数 |
>|-----|:------|-------|
>| changePosition | 当列表滚动时触发 |  { position: 滚动位置 ,height: 容器高度,width: 容器宽度,innerHeight: 列表高度,innerWidth: 列表宽度}  |


