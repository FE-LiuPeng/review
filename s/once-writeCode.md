### <center>一更新文--论手写的重要性?</center>

### Title 
- Throttle
- Febounce
- My Promise
- Flat
- My Reduce
- Curry
- Event Bus
- My New
- My Call
- My Apply
- My instanceOf
- My Constructor

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
      setTimeout(later, time)
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