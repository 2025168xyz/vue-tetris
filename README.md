## 用Vue、Vuex、Immutable做俄罗斯方块

----
本项目灵感来源于 React 版的俄罗斯方块,由于对其实现原理较感兴趣,而且相比于 React 更喜欢 Vue, 于是把 React 版的重构为了 Vue 版的,大致思路是把组件当成一个个函数,保证一个输入(props)能得到一个确定的输出(view),然后对不同方法也是做同样处理,对于 Redux 使用 Vuex 精简化

戳：[http://binaryify.github.io/vue-tetris/](http://binaryify.github.io/vue-tetris/) 玩一玩！

----
### 效果预览
![效果预览](https://img.alicdn.com/tps/TB1Ag7CNXXXXXaoXXXXXXXXXXXX-320-483.gif)

正常速度的录制，体验流畅。

### 响应式
![响应式](https://img.alicdn.com/tps/TB1AdjZNXXXXXcCapXXXXXXXXXX-480-343.gif)

不仅指屏幕的自适应，而是`在PC使用键盘、在手机使用手指的响应式操作`：

![手机](https://img.alicdn.com/tps/TB1kvJyOVXXXXbhaFXXXXXXXXXX-320-555.gif)

### 数据持久化
![数据持久化](http://7xkm8j.com1.z0.glb.clouddn.com/persistence.gif)

玩单机游戏最怕什么？断电。通过订阅 `store.subscribe`，将state储存在localStorage，精确记录所有状态。网页关了刷新了、程序崩溃了、手机没电了，重新打开连接，都可以继续。

### Vuex 状态预览（[Vue DevTools extension](https://github.com/vuejs/vue-devtools)）
![Vuex状态预览](http://7xkm8j.com1.z0.glb.clouddn.com/vuex.gif)

Vuex 设计管理了所有应存的状态，这是上面持久化的保证。

----
游戏框架使用的是 Vue + Vuex，其中再加入了 Immutable,确保性能和数据可靠性


## 1、什么是 Immutable？
Immutable 是一旦创建，就不能再被更改的数据。对 Immutable 对象的任何修改或添加删除操作都会返回一个新的 Immutable 对象。

### 初识：
让我们看下面一段代码：
``` JavaScript
function keyLog(touchFn) {
  let data = { key: 'value' };
  f(data);
  console.log(data.key); // 猜猜会打印什么？
}
```
不查看f，不知道它对 `data` 做了什么，无法确认会打印什么。但如果 `data` 是 Immutable，你可以确定打印的是 `value`：
``` JavaScript
function keyLog(touchFn) {
  let data = Immutable.Map({ key: 'value' });
  f(data);
  console.log(data.get('key'));  // value
}
```

JavaScript 中的`Object`与`Array`等使用的是引用赋值，新的对象简单的引用了原始对象，改变新也将影响旧的：
``` JavaScript
foo = {a: 1};  bar = foo;  bar.a = 2;
foo.a // 2
```
虽然这样做可以节约内存，但当应用复杂后，造成了状态不可控，是很大的隐患，节约的内存优点变得得不偿失。

Immutable则不一样，相应的：
``` JavaScript
foo = Immutable.Map({ a: 1 });  bar = foo.set('a', 2);
foo.get('a') // 1
```

### 关于 “===”：
我们知道对于`Object`与`Array`的`===`比较，是对引用地址的比较而不是“值比较”，如：
``` JavaScript
{a:1, b:2, c:3} === {a:1, b:2, c:3}; // false
[1, 2, [3, 4]] === [1, 2, [3, 4]]; // false
```
对于上面只能采用 `deepCopy`、`deepCompare`来遍历比较，不仅麻烦且好性能。

我们感受来一下`Immutable`的做法！
``` JavaScript
map1 = Immutable.Map({a:1, b:2, c:3});
map2 = Immutable.Map({a:1, b:2, c:3});
Immutable.is(map1, map2); // true

// List1 = Immutable.List([1, 2, Immutable.List[3, 4]]);
List1 = Immutable.fromJS([1, 2, [3, 4]]);
List2 = Immutable.fromJS([1, 2, [3, 4]]);
Immutable.is(List1, List2); // true
```


Immutable学习资料：
* [Immutable.js](http://facebook.github.io/immutable-js/)


## 2、Web Audio Api
游戏里有很多不同的音效，而实际上只引用了一个音效文件：[/build/music.mp3](https://github.com/Binaryify/vue-tetris/blob/master/build/music.mp3)。借助`Web Audio Api`能够以毫秒级精确、高频率的播放音效，这是`<audio>`标签所做不到的。在游戏进行中按住方向键移动方块，便可以听到高频率的音效。

![网页音效进阶](https://img.alicdn.com/tps/TB1fYgzNXXXXXXnXpXXXXXXXXXX-633-358.png)

`WAA` 是一套全新的相对独立的接口系统，对音频文件拥有更高的处理权限以及更专业的内置音频效果，是W3C的推荐接口，能专业处理“音速、音量、环境、音色可视化、高频、音向”等需求，下图介绍了WAA的使用流程。

![流程](https://img.alicdn.com/tps/TB1nBf1NXXXXXagapXXXXXXXXXX-520-371.png)

其中Source代表一个音频源，Destination代表最终的输出，多个Source合成出了Destination。
源代码：[/src/unit/music.js](https://github.com/Binaryify/vue-tetris/blob/master/src/unit/music.js) 实现了ajax加载mp3，并转为WAA，控制播放的过程。

`WAA` 在各个浏览器的最新2个版本下的支持情况（[CanIUse](http://caniuse.com/#search=webaudio)）

![浏览器兼容](https://img.alicdn.com/tps/TB15z4VOVXXXXahaXXXXXXXXXXX-679-133.png)

可以看到IE阵营与大部分安卓机不能使用，其他ok。


Web Audio Api 学习资料：
* [Web API 接口| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API)
* [Getting Started with Web Audio API](http://www.html5rocks.com/en/tutorials/webaudio/intro/)

----
## 3、游戏在体验上的优化
* 技术：
	* 按下方向键水平移动和竖直移动的触发频率是不同的，游戏可以定义触发频率，代替原生的事件频率，源代码：[/src/unit/event.js](https://github.com/Binaryify/vue-tetris/blob/master/src/unit/event.js) ；
	* 左右移动可以 delay 掉落的速度，但在撞墙移动的时候 delay 的稍小；在速度为6级时 通过delay 会保证在一行内水平完整移动一次；
	* 对按钮同时注册`touchstart`和`mousedown`事件，以供响应式游戏。当`touchstart`发生时，不会触发`mousedown`，而当`mousedown`发生时，由于鼠标移开事件元素可以不触发`mouseup`，将同时监听`mouseout` 模拟 `mouseup`。源代码：[/src/components/keyboard/index.js](https://github.com/Binaryify/vue-tetris/blob/master/src/components/keyboard/index.js)；
	* 监听了 `visibilitychange` 事件，当页面被隐藏\切换的时候，游戏将不会进行，切换回来将继续，这个`focus`状态也被写进了 Vuex 中。所以当用手机玩来`电话`时，游戏进度将保存；PC开着游戏干别的也不会听到gameover，这有点像 `ios` 应用的切换。
	* 在`任意`时刻刷新网页，（比如消除方块时、游戏结束时）也能还原当前状态；
	* 游戏中唯一用到的图片是![image](https://img.alicdn.com/tps/TB1qq7kNXXXXXacXFXXXXXXXXXX-400-186.png)，其他都是CSS；
	* 游戏兼容 Chrome、Firefox、IE9+、Edge等；
* 玩法：
	* 可以在游戏未开始时制定初始的棋盘（十个级别）和速度（六个级别）；
	* 一次消除1行得100分、2行得300分、3行得700分、4行得1500分；
	* 方块掉落速度会随着消除的行数增加（每20行增加一个级别）；

----

## 4、开发中的经验梳理
Vue 版本和 React 版本核心代码基本相同,但在编写组件的时候遇到了几个问题,比如:
1. React 版的 store  使用了 immutable 结构的数据,vuex 上的 store 如果使用了 immutable 结构,不利用监听数据变化,故把store 的数据全部使用了普通的数据,在需要这些数据的地方通过 immutable 提供的 `fromJS` 转换,在需要普通数据的地方再通过 immutable 的 `toJS` 转换成普通数据  

2. Vue 没有 React 的`componentWillReceiveProps` 的生命周期,我的解决方法是使用 watch 配合 `deep:true` 来监听 props 的变化,如:
```js
watch: {
  $props: {
    deep: true,
    handler(nextProps) {
      //xxx
    }
  }
}
```

3. `matrix 组件` 的功能逻辑较复杂,使用 `template` 模版来渲染组件已经不合适了,通过自定义 render 方法再手动触发很繁琐,我的解决方法是通过 Vue 的 jsx 转换插件[babel-plugin-transform-vue-jsx](https://github.com/vuejs/babel-plugin-transform-vue-jsx)来使用 jsx 语法对页面进行渲染,当 props 或 state 变化了自动触发 render 方法,另外要注意的是 vue 的 jsx 和 React 的 jsx 书写上有一点的差异

## 5、架构差异
Redux 的数据流向是 把 store 的状态转化为 props 注入到 根组件,根组件再把这些 props 传入不同组件,当 store 的状态变化,根组件会重新 render, 更新子组件上的 props,子组件再 根据新 props重新 render
引用知乎一个答友的回答[https://www.zhihu.com/question/47686258](https://www.zhihu.com/question/47686258)来说就是:
>单例store的数据在react中可以通过view组件的属性（props）不断由父模块**“单向”**传递给子模块，形成一个树状分流结构。如果我们把redux比作整个应用的“心肺” （redux的flux功能像心脏，reducer功能像肺部毛细血管），那么这个过程可以比作心脏（store）将氧分子（数据）通过动脉毛细血管（props）送到各个器官组织（view组件）末端的view组件，又可以通过flux机制，将携带交互意图信息的action反馈给store。这个过程有点像将携带代谢产物的“红细胞”（action）通过静脉毛细血管又泵回心脏（store）action流回到store以后，action以参数的形式又被分流到各个具体的reducer组件中，这些reducer同样构成一个树状的hierarchy。这个过程像静脉血中的红细胞（action）被运输到肺部毛细血管（reducer组件）接收到action后，各个child reducer以返回值的形式，将最新的state返回给parent reducer，最终确保整个单例store的所有数据是最新的。这个过程可以比作肺部毛细血管的血液充氧后，又被重新泵回了心脏回到步骤1

而 vuex 的思路则不同,任何组件都随时可以通过 this.$store.state.xxx 获取 store 上的数据,更自由,只要 store 上的数据变了,组件都会自动重新渲染

## 6、开发
### 安装
```
npm install
```
### 运行
```
npm run dev
```
浏览自动打开 [localhost:8080](localhost:8080)
### 多语言
在 [i18n.json](https://github.com/Binaryify/vue-tetris/blob/master/src/i18n.json) 配置多语言环境，使用"lan"参数匹配语言如：`https://Binaryify.github.io/vue-tetris/?lan=en`
### 打包编译
```
npm run build
```

在 `dist` 文件夹下生成结果。