### <center>一更新文--论手写的重要性?</center>
了解感念即可， 追溯根源并深入考虑谈何三画五笔？
### Title 
- Throttle
- UUID
- Debounce
- Flat
- 原型继承
- 构造函数继承
- 组合继承
- 寄生式组合继承
- class 继承
- 简单的AST
- 深拷贝
- My Reduce
- Curry
- Event Bus
- My New
- My Call
- My Apply
- My instanceOf
- My Constructor
- My Promise

#### Throttle
```js
const throttle = (fn, delay) => {
  let lock, args

  function later () {
    lock = false
    if (args) {
      wrapperFn.apply(context, args)
      args = false
    }
  }

  function wrapperFn () {
    if (lock) {
      args = arguments
    } else {
      lock = true
      fn.apply(context, arguments)
      setTimeout(later, delay)
    }
  }

  return wrapperFn
}

```


#### UUID
```js
const uuid = () => {
  function b(a) {
    return a ? (a ^ Math.random() * 16 >> a / 4).toString(16) : ([1e7] + -[1e3] + -4e3 + -8e3 + -1e11).replace(/[018]/g, b)
  }

  return b()
}
```

#### Debounce
```js
    const debounce = (fn, delay) => {
        let time;
        return () => {
            time && clearTimeout(time);
            time = setTimeout(() => {
                fn()
            }, delay)
        }
    }
```

#### Flat
```js
    const faltten = (arr) => {
        return arr.reduce((t, i) => {
            Array.isArray(t) ? (t=t.concat(faltten(i))) : t.push(i)
            return t
        }, [])
    }

    const flat = arr => arr.flat(Infinity);
```

#### 原型继承
```js
function Person() {
    this.arr = [1,2]
}
Person.prototype.add = () => {
    console.log('add')
}
function Son() {
    this.flag = false;
}
Son.prototype = new Person()
let _ = new Son()

// 参数没办法传递过去
```


#### 构造函数继承
```js
function Person(name) {
    this.arr = [1,2]；
    this.func1 = () => {};
    this.func2 = () => {};
}
Person.prototype.func3 = () => 3
function Son(name) {
    Person.call(this, name) // this搞过来 然后在传参数
    this.flag = false;
}
Son.prototype = new Person()
let _ = new Son('Child')
let _2 = new Son('Child2')
// 那Person里面的func次次都挂载，增大内存（总说这个增加大， 实际中， 呵呵）
```

#### 组合继承
```js
// 就是都往继承者身上怼
function Person(name) {
    this.arr = [1,2]；
    this.func1 = () => {};
    this.func2 = () => {};
}
Person.prototype.func3 = () => 3
function Son(name) {
    Person.call(this, name) // this搞过来 然后在传参数
    this.flag = false;
}
Son.prototype = new Person();
Son.prototype.constructor = Son;
 // 这下都给人家了吧， 鸡毛不剩， 把那上面的方法都扒过来， 要不然构造器指向Person 还是用人家的，，， 这样子就区分开来了， 各自独立出来， 然后还能穿参数 多牛逼，
// 但是调用了两次Person啊， 
let _ = new Son('Child')
```


#### 寄生式组合继承 
```js
// 这个 “寄生式组合继承” 听起来真牛逼。
// 上面调用了两次， 就是解决这个玩意，那调用两次咋整，再搞一个临时的构造函数呗。。。

function Person(name) {
    this.arr = [1,2]；
    this.func1 = () => {};
    this.func2 = () => {};
}
Person.prototype.func3 = () => 3
function Son(name) {
    Person.call(this, name) // this搞过来 然后在传参数
    this.flag = false;
}
// 方法1
function FuncTmp() {}
FuncTmp.prototype = Person.prototype;
let _f = new FuncTmp();
_f.constructor = Son;
Son.prototype = _f;

// 方法2
Son.prototype = Object.create(Person.prototype);// 重新搞一份出来，
Son.prototype.constructor = Son;
```

#### class 继承
```js
class Person{
    constructor(){}
}
class Son extends Person{
    constructor(){ 
        super() 
    }
}

```


#### 简单的AST
```js
function html2AST(html) {
    const startTag = /<([a-zA-Z_][\w\-\.]*)((?:\s+([a-zA-Z_:][-a-zA-Z0-9_:.]*)\s*=\s*(?:"([^"]*)"|'([^']*)'|([^\s"'=<>`]+)))*)\s*(\/?)>/;
  
    const endTag = /<\/([a-zA-Z_][\w\-\.]*)>/;
  
    const attr = /([a-zA-Z_:][-a-zA-Z0-9_:.]*)\s*=\s*(?:"([^"]*)"|'([^']*)'|([^\s"'=<>`]+))/g;
  
    const bufArray = [];
    const results = {
      node: "root",
      child: []
    };
    let chars;
    let match;
    let last;
    while (html && last != html) {
      last = html;
      chars = true; // 是不是文本内容
      if (html.indexOf("</") == 0) {
        match = html.match(endTag);
        if (match) {
          chars = false;
          html = html.substring(match[0].length);
          match[0].replace(endTag, parseEndTag);
        }
      } else if (html.indexOf("<") == 0) {
        match = html.match(startTag);
        if (match) {
          chars = false;
          html = html.substring(match[0].length);
          match[0].replace(startTag, parseStartTag);
        }
      }
      if (chars) {
        let index = html.indexOf("<");
        let text;
        if (index < 0) {
          text = html;
          html = "";
        } else {
          text = html.substring(0, index);
          html = html.substring(index);
        }
        const node = {
          node: "text",
          text
        };
        pushChild(node);
      }
    }
    function pushChild(node) {
      if (bufArray.length === 0) {
        results.child.push(node);
      } else {
        const parent = bufArray[bufArray.length - 1];
        if (typeof parent.child == "undefined") {
          parent.child = [];
        }
        parent.child.push(node);
      }
    }
    function parseStartTag(tag, tagName, rest) {
      tagName = tagName.toLowerCase();
  
      const ds = {};
      const attrs = [];
      let unary = !!arguments[7];
  
      const node = {
        node: "element",
        tag: tagName
      };
      rest.replace(attr, function(match, name) {
        const value = arguments[2]
          ? arguments[2]
          : arguments[3]
          ? arguments[3]
          : arguments[4]
          ? arguments[4]
          : "";
        if (name && name.indexOf("data-") == 0) {
          ds[name.replace("data-", "")] = value;
        } else {
          if (name == "class") {
            node.class = value;
          } else {
            attrs.push({
              name,
              value
            });
          }
        }
      });
      node.dataset = ds;
      node.attrs = attrs;
      if (!unary) {
        bufArray.push(node);
      } else {
        pushChild(node);
      }
    }
    function parseEndTag(tag, tagName) {
      let pos = 0;
      for (pos = bufArray.length - 1; pos >= 0; pos--) {
        if (bufArray[pos].tag == tagName) {
          break;
        }
      }
      if (pos >= 0) {
        pushChild(bufArray.pop());
      }
    }
    return results;
  }

  const ast = html2AST('<div>我是一个div</div>')

```

#### 深拷贝
```js
// 在满世界都有着深拷贝神仙的时候， 唉 去网上一搜， 人外又人的在钻研 性能 界点值等等这种，但是这种挣扎的本根是啥？ 单纯为面试我不言语， 工作中这种拷贝的方式很多吧， 为什么要求知与个人实现呢？ Object.create 、 解构赋值 其实还有很多， 非要刨根问底， 不见得是好事儿，  因为我是学法的，如果我对法律存有敬畏之心，那我也会变成利用BUG的人渣。

// 还有 ‘Object.prototype.hasOwnProperty.call(target, k)’ 这个， 为啥要从Object直接拿来用， 如果使用当前对象万一被人恶意修改hasOwnProperty这个方法呢？ 当然 Object也可以修改， 我笑了

// 我直接搬运大佬的吧  理解起来也很简单。 

// 保持引用关系
function cloneForce(x) {
    // =============
    const uniqueList = []; // 用来去重
    // =============

    let root = {};

    // 循环数组
    const loopList = [
        {
            parent: root,
            key: undefined,
            data: x,
        }
    ];

    while(loopList.length) {
        // 深度优先
        const node = loopList.pop();
        const parent = node.parent;
        const key = node.key;
        const data = node.data;

        // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }
        
        // =============
        // 数据已经存在
        let uniqueData = find(uniqueList, data);
        if (uniqueData) {
            parent[key] = uniqueData.target;
            break; // 中断本次循环
        }

        // 数据不存在
        // 保存源数据，在拷贝数据中对应的引用
        uniqueList.push({
            source: data,
            target: res,
        });
        // =============
    
        for(let k in data) {
            if (data.hasOwnProperty(k)) {
                if (typeof data[k] === 'object') {
                    // 下一次循环
                    loopList.push({
                        parent: res,
                        key: k,
                        data: data[k],
                    });
                } else {
                    res[k] = data[k];
                }
            }
        }
    }

    return root;
}

function find(arr, item) {
    for(let i = 0; i < arr.length; i++) {
        if (arr[i].source === item) {
            return arr[i];
        }
    }

    return null;
}


```




### 参见
- [思否牛逼的深拷贝](https://segmentfault.com/a/1190000016672263)
- [jsmini](https://github.com/jsmini/clone)