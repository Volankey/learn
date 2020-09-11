
# 使用中的新增特性个人感觉有用的

循环语句使用 v-if
```
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```

内联使用event
```
<button @click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

多handler触发
```
<!-- both one() and two() will execute on button click -->
<button @click="one($event), two($event)">
  Submit`
</button>
``` 

2.4.0 inheritAttrs 不自动继承attrs

```
与单个根节点组件不同，具有多个根节点的组件没有自动属性失效行为。如果$attrs没有显式绑定，则会发出运行时警告。

// This will raise a warning
app.component('custom-layout', {
  template: `
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  `
})

// No warnings, $attrs are passed to <main> element
app.component('custom-layout', {
  template: `
    <header>...</header>
    <main v-bind="$attrs">...</main>
    <footer>...</footer>
  `
})
```

事件可以注册了

emits:['submit']
还可以添加验证函数
```
  emits: {
    // No validation
    click: null,

    // Validate submit event
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  },
```

## v-model自定义组件使用

支持多个


```
const app = Vue.createApp({})

<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
  v-model="value"
></user-name>
const app = Vue.createApp({})

app.component('user-name', {
  props: {
    firstName: String,
    lastName: String,
     modelValue:String,
  },
  template: `
    <input 
      type="text"
      :value="firstName"
      @input="$emit('update:firstName', $event.target.value)">

    <input
      type="text"
      :value="lastName"
      @input="$emit('update:lastName', $event.target.value)">
    <input
      type="text"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)">
  `
})


const HelloVueApp = {
  components: {
    UserName,
  },
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe',
      value:''
    };
  },
};
```


provide inject 响应式
```
app.component('todo-list', {
  // ...
  provide() {
    return {
      todoLength: Vue.computed(() => this.todos.length)
    }
  }
})
```

v-once 保证只被计算一次


v-if可以添加class来进行动画

```
Class-based Animations & Transitions
Though the <transition> component can be wonderful for components entering and leaving, you can also activate an animation without mounting a component, by adding a conditional class.

<div id="demo">
  Push this button to do something you shouldn't be doing:<br />

  <div :class="{ shake: noActivated }">
    <button @click="noActivated = true">Click me</button>
    <span v-if="noActivated">Oh no!</span>
  </div>
</div>

.shake {
  animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
  transform: translate3d(0, 0, 0);
  backface-visibility: hidden;
  perspective: 1000px;
}

@keyframes shake {
  10%,
  90% {
    transform: translate3d(-1px, 0, 0);
  }

  20%,
  80% {
    transform: translate3d(2px, 0, 0);
  }

  30%,
  50%,
  70% {
    transform: translate3d(-4px, 0, 0);
  }

  40%,
  60% {
    transform: translate3d(4px, 0, 0);
  }
}
```



### render
```
v-model
The v-model directive is expanded to modelValue and onUpdate:modelValue props during template compilation—we will have to provide these props ourselves:

props: ['modelValue'],
render() {
  return Vue.h(SomeComponent, {
    modelValue: this.modelValue,
    'onUpdate:modelValue': value => this.$emit('update:modelValue', value)
  })
}
```
Event Modifiers
For the .passive, .capture, and .once event modifiers, Vue offers object syntax of the handler:

For example:

render() {
  return Vue.h('input', {
    onClick: {
      handler: this.doThisInCapturingMode,
      capture: true
    },
    onKeyUp: {
      handler: this.doThisOnce,
      once: true
    },
    onMouseOver: {
      handler: this.doThisOnceInCapturingMode,
      once: true,
      capture: true
    },
  })
}

slot传递
```
return Vue.h('h1', [
      Vue.h(
        'a',
        {
         
        },
        this.$slots.default(),
        Vue.h('div',{},this.$slots.another && this.$slots.another()),
      )
    ])
 return (
      Vue.h(AnchoredHeading,{},{default:()=>'asd1',another:()=>'sss'})
    )
```


# proxy

```
Vue在内部跟踪所有被激活的对象，因此它总是为相同的对象返回相同的代理。当从反应性代理访问嵌套对象时，该对象在返回之前也会被转换为代理
const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      return reactive(value)
    } else {
      return value
    }
  }
  // ...
}
```

refs()可以是非对象变为响应式,但有一些需要注意的点,感觉有点坑
https://v3.vuejs.org/guide/reactivity-fundamentals.html#ref-unwrapping

readonly是响应式对象不可以变化

解构会让reactive的对象失去响应性使用toRef链接
```
Destructuring Reactive State
When we want to use a few properties of the large reactive object, it could be tempting to use ES6 destructuring to get properties we want:

import { reactive } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = book
Unfortunately, with such a destructuring the reactivity for both properties would be lost. For such a case, we need to convert our reactive object to a set of refs. These refs will retain the reactive connection to the source object:

import { reactive, toRefs } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = toRefs(book)

title.value = 'Vue 3 Detailed Guide' // we need to use .value as title is a ref now
console.log(book.title) // 'Vue 3 Detailed Guide'
```

解构props会失去响应性解决方法如上
```
 setup(props,context) {
    console.log(props.title)
  }
```