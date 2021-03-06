# 2019

## 一月

### 利用 Coverage 检测可以懒加载的 modules

1、打开 devTools，按`Ctrl+shift+p`，mac(`cmd+shift+p`)，输入`Coverage`，选`Drawer: Coverage`

2、reload

3、可以看到哪些 modules 可以用`import()`懒加载了

### nginx vue history 爬坑

按照官方`nginx`的参考配置：

```bash
location / {
  try_files $uri $uri/ /index.html;
}
```

如果是项目在根目录倒没啥问题，但如果项目在 xxx 路径下，比如在`http://ip/vue/`路径下，点击跳转到路由`http://ip/vue/about`下是 ok 的，但是一刷新页面，你会发现就不好使了。原因很简单，就在上面的配置中:

`try_files $uri $uri/ /index.html` => `http://ip/vue/about/index.html`

所以，这种情况正确的操作是：

```bash
location /vue/ {
  try_files $uri $uri/ /vue/index.html;# 全部跳回到vue/index.html页面中
}
```

注意， `/vue/`实际上你上面配的`root`下的 vue 文件夹，比如你的`root`是`/app`，`location /vue/`即为 `location /app/vue/`

### iview menu 组件无法高亮展开问题

在 iview Menu 组件中，如果数据是异步请求的，动态设置`activeName`、`openNames`会不生效，原因是 activeName、或者 openNames 这些优先告诉 Menu 组件了，挂载时(menuData 还没获取到)就已经确定好这些状态，就算是 menuData 获取到了，也不会触发 setter 进行页面状态更新

**解决方案**

方案 1：

```js
// 利用watch
watch() {
  // 异步获取数据更新时，需要进行高亮、展开节点更新
  menuData() {
    this.$nextTick(() => {
      this.$refs.menu.updateActiveName();
      this.$refs.menu.updateOpened();
    });
  },
  activeName(value) {
    this.$nextTick(() => {
      this.$refs.menu.updateActiveName();
    });
  },
  openNames(value) {
    this.$nextTick(() => {
      this.$refs.menu.updateOpened();
    });
  }
}
```

方案 2：

`<Menu v-if="menuData.length !== 0"/>`

### 滚动进度条核心代码

::: tip 原理
视口滚动的距离 / 文档总高度 - 视口高度
:::

```js
// jq
($(window).scrollTop() / ($(document).height() - $(window).height())) * 100;

// js
(window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100;
```

## 二月

### vue jsx 使用全局组件

在 vue jsx 中只能使用`(kebab-case)`的全局组件，不能使用`(PascalCase)`，如果要使用`(PascalCase)`，可以这么写

```js
render (h) {
  const { Tabs, TabPane } = this.$options.components;
  return (
    <Tabs>
      <TabPane></TabPane>
    </Tabs>
  )
}
```

### tomcat 开启 gzip 配置

在`conf/server.xml`文件中进行配置

```xml
<Connector  port="8088"
            protocol="HTTP/1.1"
            connectionTimeout="20000"
            redirectPort="8443" URIEncoding="utf-8"
	          useSendfile="false"
            compression="on"
            compressionMinSize="50"
            noCompressionUserAgents="gozilla,traviata"
            compressableMimeType="text/html,text/xml,text/javascript,application/x-javascript,application/javascript,text/css,text/plain"
/>
```

## 三月

### vue v-model 细节

`v-model` 在内部使用不同的属性为不同的输入元素并抛出不同的事件：

- text 和 textarea 元素使用 value 属性和 input 事件
- checkbox 和 radio 使用 checked 属性和 change 事件
- select 字段将 value 作为 prop 并将 change 作为事件

### js 找出数组最大最小值

```js
const getMaxNumber = array => Math.max(...array);
const getMinNumber = array => Math.min(...array);
```

### vue 强迫重新渲染

```js
Vue.component("comp", {
  created() {
    console.log("被重新渲染了");
  },
  render(h) {
    return h("span", "组件");
  }
});
const app = new Vue({
  el: "#app",
  data: {
    key: 0
  },
  methods: {
    update() {
      this.key++;
    }
  },
  render(h) {
    const vm = this;
    return h("div", [
      h("comp", {
        key: vm.key
      }),
      h(
        "button",
        {
          on: {
            click: function() {
              vm.update();
            }
          }
        },
        "刷新"
      )
    ]);
  }
});
```

### vue 路由跳转重渲染

比如，在点击菜单切换，在父组件中会进行`$router.push({name: xxx, query: xxx})`切换子路由页面(同一个组件)，子路由页面会获取相关路由参数进行数据请求，但是 vue-router 会默认**同一组件不重复实例化**，所以我们有 2 种方法：

1、子路由`watch: $route`，然后进行相关请求逻辑

2、每次跳转路由(即使是同一个)都强制重刷新

第一种方法经常用了，这里不再详叙。根据上个笔记的启发，我们可以通过如下写法轻松实现强制组件重渲染

```html
<router-view :key="$route.fullPath" />
```

注意，组件渲染量少的话可以使用，如果组件很重，还是建议使用第一种方法

## 四月

### echarts 平均值 toolTip 的问题

在 echarts 默认出现的 toolTip 中，可能会出现`null`等奇怪的东西出来，需要用 formatter 解决

```js
const option = {
  tooltip: {
    formatter(params) {
      // marker 是小圆点
      const { color, marker, name, value } = params;
      if (typeof color === "string") {
        return `${marker}${name}: ${value}`;
      }
      return `${name}: ${value}`;
    }
  }
};
```

### vue 中 self 事件修饰符

`.self` 事件修饰符实际等于 `if (event.target !== event.currentTarget) return`

### 正则过滤 "./index.js" 文件

`/\.\/(?!index\.)\w*\.js$/.test("./index.js")`

### form 表单中 button 的坑

在`form`表单中，如果含有`button`按钮标签，点击则会重载页面，原因是因为在浏览器中，`button`的`type`属性会有默认值，w3c 标准如下：

> 请始终为按钮规定 type 属性。Internet Explorer 的默认类型是 "button"，而其他浏览器中（包括 W3C 规范）的默认值是 "submit"。

`type`属性为`submit`时会在表单中触发重载页面，所以指定 `type` 属性为 `button`即可

在 form 表单中，应该用`input`标签来创建按钮

### cached 函数

该函数一般用于**缓存比较耗时**的函数，主要利用闭包进行实现

```js
function cached(fn) {
  const cache = Object.create(null);
  return function cachedFn(str) {
    var hit = cache[str];
    return hit || (cache[str] = fn(str));
  };
}

const fn = cached(function(str) {
  let i = 10000;
  while (i--) {}
  return str;
});

console.time("test1");
fn(1); // 2.9ms
console.timeEnd("test1");

console.time("test2");
fn(1); // 0.005ms // 第一次被缓存，所以非常快
console.timeEnd("test2");
```

### yarn 升级指定包

yarn upgrade-interactive --latest

### 关于 vue button 组件的小坑

自己封装了一个`button`组件

```js
const button = {
  name: 'Button',
  methods: {
    handleClick() {
      this.$emit('click')
    }
  }
  render(h) {
    return h('button', {
      on: {
        click: () => {
          this.handleClick();
        }
      }
    }, this.$slots.default)
  }
}
```

使用

<Button @click.stop.prevent="handleClick">按钮</Button>

这种情况下，是会报找不到 Cannot read property 'stopPropagation' of undefined 这种错

解决方案是 把 event 对象传出去即可

```js
render(h) {
    return h('button', {
      on: {
        click: (event) => {
          this.handleClick(event);
        }
      }
    }, this.$slots.default)
  }
```

## 五月

### let、const、function、class 在 switch 中作用域问题

::: tip 原理
词法声明在整个 switch 语句块中是可见的，但是它只有在运行到它定义的 case 语句时，才会进行初始化操作
:::

错误示例：

```js
switch (foo) {
  case 1:
    let x = 1;
    break;
  case 2:
    const y = 2;
    break;
  case 3:
    function f() {}
    break;
  default:
    class C {}
}
```

正确示例：

```js
switch (foo) {
  case 1: {
    let x = 1;
    break;
  }
  case 2: {
    const y = 2;
    break;
  }
  case 3: {
    function f() {}
    break;
  }
  case 4:
    var z = 4;
    break;
  default: {
    class C {}
  }
}
```

<ToTop/>
