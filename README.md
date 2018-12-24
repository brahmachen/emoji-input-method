# emoji-input-method

微博表情输入法 [演示地址](https://brahmachen.github.io/emoji-input-method/dist/#)

## Blog

最近要做一个用于表情选择输入的表情输入法，设计稿如图所示
![设计稿](https://brahmachen.github.io/my-image-server/18-12/20181224104204.png)

整个模块区域大概分为两部分，一个是上部的文字输入提交区，用于评论内容显示；第二个是下部的表情选择区，用于展示和选择表情。当选择某个表情时，要以字符串拼接的方式把代表这个表情的字符串插入到光标的位置。下部分表情选择可以点击选择不同的类别，另外超出时可以左滑右滑分页。再就是表情数据都是以微博的表情作为素材，每个表情对应的字符串也和微博的一致。

作为一个比较简单的表情输入法，比起市面上的一些高大上的同类算是比较简洁了。这里记录一下实现的时候感觉稍微复杂的地方，以及实现过程中遇到的问题。

### 如何实现样式布局

表情分页部分的样式主要是由内外两层实现。思路是外层固定100%宽度，内层根据表情的数量计算页数，从而计算出宽度，这样在超出外层宽度时可以横向滚动。切换页数时更改内层的margin-left值可以显示相应页的内容。同时给内层的margin-left设置过度效果，从而在切页时具备了动画的能力。

每页中表情的排列显示可以使用flex布局搞定。对容器固定宽度和高度，使用`flex-wrap: wrap`实现表情元素超出容器宽度时进行换行，使用`align-content: flex-start`实现在当前页的表情数量在不足一页时依然紧凑排列。

css样式代码如下

```less
.scroll-wrapper { // 分页区域外层
    width: 100%;
    height: 157px;
    border-top: solid 1px #DDDDDD;
    position: relative;
    .face-content { // 滚动区域
        height: 100%;
        display: flex;
        transition: margin-left .5s;
        .face-page { // 单个表情页
            width: 100%;
            height: 140px;
            display: flex;
            flex-wrap: wrap;
            align-content: flex-start;
            a { // 表情
                width: 14.28%;
                height: 33.33%;
                display: flex;
                justify-content: center;
                align-items: center;
            }
        }
    }
}
```

### 数据组织与页面渲染

秉承着数据驱动的思想，可以减少dom操作，提高页面性能，优化代码结构。原始数据是一个树形结构的json串，由于拿到的表情原始数据并没有给数据分页，首先需要给数据分页。然后需要维护一个索引，代表当前激活的表情分组，当切换表情分组时修改这个索引并重新计算分页。维护另外一个索引，代表当前页数，切换表情分组时，重置这个索引回到首页，切换页数时，修改这个索引同步至相应的页数。维护另外一个值代表滚动区域应该滚到的位置，当切换页数时更新这个值，从而将页面渲染到正确的位置。

html模板代码如下（使用vue实现）：
```html
<div class="face-content"
    @touchmove.prevent
    ref="faceContent"
    :style="{ width: scrollWidth + 'px', marginLeft: -scrollPosition + 'px' }"
    >
    <div class="face-page"
        v-for="(page, index) in faces"
        :key="index">
        <a v-for="(item, j) in page"
        :key=j
        >
        <img @click="onFaceClick(`[${item.title}]`)" :src=item.src width="20" height="20">
        </a>
    </div>
</div>
```

部分js代码：

```js
computed: {
    // 切换分组时更新分页数据
    faces: function () {
        const { faceData, faceIndex } = this
        const list = faceData[faceIndex].faceList
        let n = 0
        let listGrouped = []
        while (n * 21 < list.length) {
        listGrouped.push(list.slice(n * 21, (n + 1) * 21))
        n += 1
        }
        return listGrouped
    },
    // 计算横向滚动区域的宽度
    scrollWidth: function () {
        return this.faces.length * document.body.scrollWidth
    },
    // 根据当前页数计算margin-left
    scrollPosition: function () {
        return this.active * document.body.scrollWidth
    }
},
```

### 如何实现左右滑翻页

一种比较简单的做法是当一次touch事件周期结束后比较开始和结束的位置，可以知道是向哪个方向滑动的，从而切换页数。缺点是滑动时页面不会跟手，滑动结束后才会触发切页效果。由于产品要求不高，这样的做法是被接受的，这省去了很大的麻烦。

翻页代码：

```js
const el = this.$refs.faceContent
let x1 = 0
let x2 = 0
el.ontouchstart = (evt) => {
    x1 = evt.targetTouches[0].pageX
}
el.ontouchend = (evt) => {
    x2 = evt.changedTouches[0].pageX
    if (x2 - x1 > 50) {
        this.active !== 0 && this.active--
    } else if (x2 - x1 < -50) {
        this.active !== this.faces.length - 1 && this.active++
    }
}
```

另外在测试中这里发现一个问题就是在某些国产浏览器中存在左滑返回上一个网页的默认操作。这里只需要在滚动区域的节点添加对touchmove事件阻止默认操作即可。如果使用vue的话使用内置事件修饰符`@touchmove.prevent`即可。

### 如何将表情对应的字符串插入文本框的光标位置

直接上代码：
```js
handleFaceChoose (face) {
    const value = this.shareText
    const el = this.$refs.shareTextArea
    const startPos = el.selectionStart // 获取光标的开始位置
    const endPos = el.selectionEnd // 获取光标的结束位置
    const newValue = // 拼接
        value.substring(0, startPos) +
        face +
        value.substring(endPos, value.length)
    this.shareText = newValue
    // 重置光标位置紧跟在表情文本后面
    if (el.setSelectionRange) {
        setTimeout(() => {
            const index = startPos + face.length
            el.setSelectionRange(index, index)
        }, 0)
    }
},
```
### 源码地址：[点击打开](https://github.com/brahmachen/emoji-input-method) （欢迎star）
### 演示地址：[点击打开](https://brahmachen.github.io/emoji-input-method/dist/#)
## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn start
```

### Compiles and minifies for production
```
yarn run build
```

### Run your tests
```
yarn run test
```

### Lints and fixes files
```
yarn run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
