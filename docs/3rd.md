# 09_监视属性

## 天气案例-基础语法

```xml
<!-- 绑定事件的时候 @xxx="yyy" yyy可以写一些简单的语句(如method回调只有一行时) -->
<button @click="isHot = !isHot">切换天气</button>
<button @click="toggleWeather">切换天气</button>

<script>
  ...
            methods: {
                // 若 method 只有一行代码，则建议直接将那一句语句写在标签属性里
                toggleWeather() {
                    this.isHot = !this.isHot;
                    this.timer++;
                }
            },
  ...
</script>
```

## 天气案例-监视属性

### 监视属性 watch

1. 当被监视的属性变化时，回调函数自动调用
2. 监视的属性必须存在，才能进行监视
3. 可以监视 data computed 中的属性
4. 监视的两种写法
(1).new Vue 时传入 watch 配置

(2).通过 vm.$watch 监视

#### 监视写法 1

```javascript
        const vm = new Vue({
            el: '#root',
            data: {
                isHot: true,
            },
            methods: {
                toggleWeather() {
                    this.isHot = !this.isHot;
                }
            },
            computed: {
                info() {
                    return this.isHot ? '炎热' : '凉爽';
                }
            },

            // 1.监视方法(1)
            watch: {  // 可以监视 data 和 computed 中的值
                // 'info'  对象中的 key 可以不写引号
                info: {
                    immediate: true,  // 初始化时就调用 handler   --isHot从 true 修改为了 undefined
                    // 当 info 发生改变时 handler 调用
                    handler(newValue, oldValue) {
                        console.log('info', newValue, '修改为了', oldValue);
                    }
                }
            },
        })
```
#### 监视写法 2

```javascript
        const vm = new Vue({
            el: '#root',
            data: {
                isHot: true,
            },
            methods: {
                toggleWeather() {
                    this.isHot = !this.isHot;
                }
            },
            computed: {
                info() {
                    return this.isHot ? '炎热' : '凉爽';
                }
            },
        })

        // 2.监视方法(2)
        vm.$watch('info', {
            immediate: true,  // 初始化时就调用 handler   --isHot从 true 修改为了 undefined
            // 当 info 发生改变时 handler 调用
            handler(newValue, oldValue) {
                console.log('info', newValue, '修改为了', oldValue);
            }
        })
```
当在声明 vue 实例时就确定了需要监视的属性，则用 第一种写法。
当是后续增加的监视属性，则用 第二种 写法

## 深度监视

当需要监视的属性是 如对象的数据类型时：

```xml
    <div id="root">
        <h3>a的值是: {{numbers.a}}</h3>
        <h3>b的值是: {{numbers.b}}</h3>
        <button @click="numbers.a++">点我让a+1</button>
    </div>

    <script>
        const vm = new Vue({
            el: '#root',
            data: {
                numbers: {
                    a: 11,
                    b: 22
                }
            },
            watch: {
                // **监视多级结构中某个属性的变化
                'numbers.a': {
                    handler(newValue, oldValue) {
                        console.log('numbers.a', newValue, '修改为了', oldValue);
                    }
                }
            },
        })
    <script/>
```
这时候如果 该对象 内的属性很多，是不方便监视的。
因此利用深度监视

```javascript
                // 监视多级结构中所有属性的变化
                numbers: {
                    deep: true,  // 开启深度监视
                    handler(newValue,oldValue) {
                        console.log('numbers', newValue, '修改为了', oldValue);
                    }
                }
```
### 总结

深度监视：

            (1).Vue中的watch默认不检测对象内部值得改变(一层)

            (2).配置 deep: true 可以监测对象内部值改变(多层)        

备注：

            (1).Vue自身可以监测对象内部值得改变(eg.修改numbers.a后模板重新加载)，但Vue提供的watch默认不可以

            (2).使用watch时根据数据的具体结构，界定是否采用深度监视

## 监视属性简写

### 在 new Vue 中定义

当不考虑设置 immediate deep 属性时

```javascript
watch: {
  isHot(newValue, oldValue) {
    console.log('isHot', oldValue, '修改为了', newValue);
  }
}
```

### 利用 $watch 定义

同样不考虑设置 immediate deep 属性时

```javascript
vm.$watch('isHot', function(newValue, oldValue) {
  console.log('isHot', oldValue, '修改为了', newValue);
})
```

## computed 和 watch 对比

### watch 实现姓名拼接

```xml
    <div id="root">
        姓：<input type="text" v-model="firstName" />
        <br />
        名：<input type="text" v-model="lastName" />
        <br />
        全名：<span>{{fullName}}</span>
    </div>

    <script>
        Vue.config.productionTip = false;

        const vm = new Vue({
            el: '#root',
            data: {
                firstName: '张',
                lastName: '三',

                fullName: '张-三'
            },
            watch: {
                firstName(newValue, oldValue) {
                    setTimeout(() => {  // 这里必须用箭头函数，因为箭头函数无 this，往外找 找到 fullName，fullName 的 this 是 vm
                        this.fullName = newValue + '-' + this.lastName;
                    }, 1000);
                },
                lastName(newValue, oldValue) {
                    this.fullName = this.firstName + '-' + newValue;
                }
            }
        })
    </script>
```
### computed 实现姓名拼接

```xml
    <div id="root">
        姓：<input type="text" v-model="firstName" />
        <br />
        名：<input type="text" v-model="lastName" />
        <br />
        全名：<span>{{fullName}}</span>
    </div>

    <script>
        const vm = new Vue({
            el: '#root',
            data: {
                firstName: '张',
                lastName: '三',
            },
            computed: {
                // 2.简写  
                // --一般不需要set。只考虑 读取 不考虑 修改 时，可以用下面的简写形式
                fullName: function() {
                    console.log('<<- fullName的get被调用了 ->>');
                    return this.firstName + '-' + this.lastName;
                }
            },
        })
    </script>
```
当都能实现时，一般用 computed，更方便。
注意：computed 中不能开启异步任务，watch可以

其中若在 watch 中调用 定时器，必须用箭头函数，因为箭头函数 无 this ，因此向外查找时找到 firstName 的 this (即 Vue)

### 总结

computed 和 watch 之间的区别：

1. computed 能完成的功能，watch 都能完成。
2. watch 能完成的功能，computed 不一定能完成，例如：watch 可以进行异步操作
两个重要小原则：

1. 要被 Vue 管理的函数，最好写成普通函数，this 才能是 vm 或是组件实例对象。
2. 所有不被 Vue 管理的函数(定时器回调函数，ajax回调函数，Promise的回调函数等)，最好写成箭头函数，this 才能是 vm 或是组件实例对象。
# 10_绑定样式

## class样式

### 字符串写法

适用于：样式名不确定，需要动态指定

```xml
<div class="basic" :class="mood" @click="changeMood">{{name}}</div>
<script>
    new Vue({
        el: '#root',
        data: {
            name: 'gyz',
            mood: 'normal',
        },
        methods: {
            changeMood() {
                const arr = ['normal', 'happy', 'sad'];
                let randomNum = Math.floor((Math.random() * (3 - 0) + 0));
                this.mood = arr[randomNum];               
            }
        }
    })
</script>
```

### 数组写法

适用于：要绑定的样式个数不确定，名字也不确定

以后可以用数组地 shift() push() 等方法动态修改数组数据

```xml
<div class="basic" :class="classArr">{{name}}</div>
<script>
    new Vue({
        el: '#root',
        data: {
            name: 'gyz',
            classArr: ['gyz1', 'gyz2', 'gyz3'],
        }
    })
</script>
```
上述代码会被解析为：
![图片](data:image/png;base64,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)

### 对象写法

适用于：要绑定的样式个数确定，名字也确定，但动态地决定用不用

动态修改 key 的值 (true/false

```xml
<div class="basic" :class="classObj">{{name}}</div>
<script>
    new Vue({
        el: '#root',
        data: {
            name: 'gyz',
            classObj: {
                gyz1: false,
                gyz2: true
            }
        }
    })
</script>
```
上述代码会被解析为：
![图片](data:image/png;base64,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)

Vue控制台：

![图片](data:image/png;base64,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)

## style样式

### 对象写法

```xml
<div class="basic" :style="{fontSize: fSize+'px'}">{{name}}</div>
<div class="basic" :style="{'font-size': fSize+'px'}">{{name}}</div>
<script>
    new Vue({
        el: '#root',
        data: {
            name: 'gyz',
            fSize: 40
        }
    })
</script>
```
其中 fSize 是 data 中的属性，可改变
其中对象中的 key 为css语句的'带引号'*驼峰命名*的属性  

eg. font-size -> fontSize or 'font-size'

也可以传一个对象名

```xml
<div class="basic" :style="styleObj">{{name}}</div>
<script>
    new Vue({
        el: '#root',
        styleObj: {
            // 'font-size': 50 + 'px'
            fontSize: '50px'
        }
    })
</script>
```

### 数组写法

:style="[a,b]" 其中 a, b 都是样式对象

```xml
<div class="basic" :style="[styleObj, styleObj2]">{{name}}</div>
<script>
    new Vue({
        el: '#root',
        styleObj: {
            // 'font-size': 50 + 'px'
            fontSize: '50px'
        },
        styleObj2: {
            // backgroundColor: 'grey'
            'background-color': 'grey'
        }
    })
</script>
```

## 总结

1. class 样式
            写法: class="xxx" xxx可以是字符串、对象、数组

                字符串写法，适用于：样式名不确定，需要动态指定

                数组写法，适用于：要绑定的样式个数不确定，名字也不确定

                对象写法，适用于：要绑定的样式个数确定，名字也确定，但动态决定用不用

2. style 样式
            :style="{fontSize: xxx}" 其中 xxx 是动态值

            :style="[a,b]" 其中a, b是样式对象

样式对象的 key 必须是存在的属性，否则无效

一般多用 class 样式

# 11_条件渲染

## v-show

```xml
<h2 v-show="false">欢迎来到{{name}}!</h2>
```
v-show 通过 display 设置为 none 来不显示
![图片](data:image/png;base64,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)

可传表达式

```xml
<h2 v-show="1 === 1">欢迎来到{{name}}!</h2>
```


## v-if

```xml
<button @click="n++">点我 n 值加一</button>
<div v-show="n === 1">Angular</div>
<div v-show="n === 2">React</div>
<div v-show="n === 3">Vue</div>

<div v-if="n === 1">Angular</div>
<div v-if="n === 2">React</div>
<div v-if="n === 3">Vue</div>
```
上面的代码段，v-show 和 v-if 效果相同
n 每一次值变化，每一个用到 n 的地方都会重新解析；上面用 v-show 和 v-if 都会多次加载，

但 v-show 效率更高，因为 v-if 会频繁操纵 DOM 元素

条件组

```xml
<div v-if="n === 1">Angular</div>
<div v-else-if="n === 1">React</div>
<div v-else-if="n === 3">Vue</div>
<div v-else>摆烂</div>
```
条件组必须满足标签相邻
按顺序做判断，一个满足条件后，就不往下判断了(即使后面条件成立)

v-if 必须在判断组中作为第一个，否则会报错

### 
### 配合 template 批量条件判断

```xml
<!-- 
    使用 template 标签，template 不会破坏结构，不会出现在页面中
    *** 只能配合 v-if 使用
-->
<template v-if="n === 1">
    <h2>hello 1</h2>
    <h2>hello 2</h2>
    <h2>hello 3</h2>
</template>
```
template 只能配合 v-if 使用，v-show 无效
## 总结

1. v-if
            写法：

                (1) v-if="表达式"

                (2) v-else-if="表达式"

                (3) v-else

            适用于：切换频率较低的场景

            特点：不展示 DOM 元素 直接被移除

            注意：v-if 可以和 v-else-if, v-else 一起使用，要求结构不能被打断。必须相邻

2. v-show
            写法：v-show="表达式"

            适用于：切换频率较高的厂家

            特点：不展示的 DOM 元素未被移除，仅仅是 display: none

            

3. 备注：使用 v-if 时，元素可能无法获取到，而用 v-show 一定可以获取到

