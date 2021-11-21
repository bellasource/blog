---
title: MVVM及vue双向数据绑定分析
date: 2020/08/28
tags: ["vue"]
---

## MVVM 和 MVC 区别

### VVM

1）M：数据，V:视图，VM：负责连接 view 和 model，在 vue 中即 vue 实例对象
2）view 和 model 无直接联系，vue 通过双向数据绑定实现 view 和 model 的自动同步更新，不需要手动操作 DOM

### MVC

1）用户对 view 的操作交给 controll 处理，controll 响应 view 事件调用 model 的接口，对数据进行处理，一旦 model 发生变化，将 view 视图更新
2）在 react 中，c 是 setState

---

## vue 双向数据绑定

### 数据代理

将 data 对象的所有属性添加到 vue 实例对象上（vm）。
在实例化时，通过 **Object.keys** 获取 data 对象的所有属性，遍历并通过 **Object.defineProperty()** 给 vm 添加 data 中的所有属性，并设置 get 和 set 方法。（vm 对象中 未直接添加 data 属性）

```javascript
function MVVM(options) {
  var data = (this._data = this.$options.data);
  // 仅遍历data的属性
  Object.keys(data).forEach(function (key) {
    me._proxyData(key);
  });
}

MVVM.prototype = {
  constructor: MVVM,
  _proxyData: function (key, setter, getter) {
    var me = this;
    setter =
      setter ||
      Object.defineProperty(me, key, {
        configurable: false,
        enumerable: true,
        get: function proxyGetter() {
          return me._data[key]; //读取属性，会进入数据劫持data对象的get方法中
        },
        set: function proxySetter(newVal) {
          me._data[key] = newVal;
        },
      });
  },
};
```

### 数据劫持

构建劫持对象，将 data 保存在劫持对象的 data 属性中，递归遍历 data 中所有属性，给每个属性构建唯一的 Dep 对象，并通过 Object.defineProperty 方法将属性添加到劫持对象的 data 属性中，此时：**一个属性对应一个 Dep 对象**

```js
Observer.prototype = {
  constructor: Observer,
  defineReactive: function (data, key, val) {
    var dep = new Dep();
    var childObj = observe(val);
    Object.defineProperty(data, key, {
      enumerable: true,
      configurable: false,
      get: function () {
        if (Dep.target) {
          dep.depend();
        }
        return val;
      },
      set: function (newVal) {
        if (newVal === val) {
          return;
        }
        val = newVal;
        // 新的值是object的话，进行监听
        childObj = observe(newVal);
        // 通知订阅者
        dep.notify();
      },
    });
  },
};
Dep.prototype = {
  addSub: function (sub) {
    this.subs.push(sub); //将watcher添加在dep属性中
    this.depIds[dep.id] = dep; //将dep{dep对象id：dep对象}添加到watcher的属性中
  },

  depend: function () {
    // Dep.target保存的是当前的watcher监视对象,this为dep对象；给watcher添加dep劫持对象
    Dep.target.addDep(this);
  },

  removeSub: function (sub) {
    var index = this.subs.indexOf(sub);
    if (index != -1) {
      this.subs.splice(index, 1);
    }
  },

  notify: function () {
    this.subs.forEach(function (sub) {
      //更新模板数据
      sub.update();
    });
  },
};
```

### 模板解析

1）**createDocumentFragment**创建文档碎片对象，将原生节点内容拷贝到文档碎片对象中;
2）遍历文档对象子节点，判断如果文本节点并且为符合插值语法的，将文本节点内容 textContent 替换；如果是标签节点：获取标签属性 node.attributes，匹配 v-text,v-html,:class,v-on 等指令，并给标签添加对应属性(node.className,node.innerHtml,node.textContent,addEventListener),最后移出该属性 removeAttribute，
3）替插值语法或换指令等后，构建与其对应的唯一 watcher 监视对象，并建立 watcher 和 dep 之间的联系，此时:**模板中一个表达式对应一个 watcher 对象**
4）如果标签有子节点，递归操作（源码中调用 methods 中的方法时，通过 bind 将 this 指向了实例对象）
5）主线：在 template 中获取属性值——》调用 MVVM 中属性的 get 方法——》调用 Observer 的 get 方法——》dep.depend()——》watcher.addDep(dep): 将 watcher 对象 push 到 dep 属性中，将 id:dep 键值对添加到 watcher 属性 depIds 中

```js
function Watcher(vm, expOrFn, cb) {
  this.cb = cb;
  this.vm = vm;
  this.expOrFn = expOrFn;
  this.depIds = {};

  if (typeof expOrFn === "function") {
    this.getter = expOrFn;
  } else {
    this.getter = this.parseGetter(expOrFn.trim());
  }
  this.value = this.get();
}

Watcher.prototype = {
  constructor: Watcher,
  update: function () {
    this.run();
  },
  run: function () {
    var value = this.get();
    var oldVal = this.value;
    if (value !== oldVal) {
      this.value = value;
      this.cb.call(this.vm, value, oldVal);
    }
  },
  addDep: function (dep) {
    if (!this.depIds.hasOwnProperty(dep.id)) {
      dep.addSub(this); //将watcher对象添加至dep对象sub属性中
      this.depIds[dep.id] = dep;
    }
  },
  get: function () {
    Dep.target = this; //将watcher监视对象赋值给劫持对象的target
    var value = this.getter.call(this.vm, this.vm); //获取属性值
    Dep.target = null;
    return value;
  },
  parseGetter: function (exp) {
    if (/[^\w.$]/.test(exp)) return;
    var exps = exp.split(".");

    return function (obj) {
      for (var i = 0, len = exps.length; i < len; i++) {
        if (!obj) return;
        obj = obj[exps[i]];
      }
      return obj;
    };
  },
};
```

### 数据更新界面渲染流程

主线：修改 data 数据——》判断为对象，劫持-》mvvm 中的 set 方法——》observer 的 set 方法中：调用 dep.notify()——》方法内部获取当前 dep 的所有 watcher，进行数据更新，模板解析，更新界面

### 小结

双向数据绑定原理及通过 Object.defindeProperty 中 get 和 set 方法，当读取数据时，自动调用 get 方法，当修改数据时，自动调用 set 方法，检测到数据变化，通过 dep 通知所有相关 watcher，自动触发重新渲染，生成新的虚拟 DOM。
内部根据 Diff 算法规则遍历并对新旧 DOM 对比，记录，最后将所有不同点，局部修改到真实 DOM 树。
