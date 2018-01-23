> compose function

##### 假设
假如有一个需求，需要把firstname和lastname组合上hello,再所有的字符串大写。那么，就是 ++"HELLO FIRSTNAME LASTNAME"++

那么用compose的概念，就是compose(toUpper, joinHello)

```
const joinHello = (first, last) => `hello ${first}, ${last}`
const toUpper = (str) => str.toUpperCase()

const fns = compose(toUpper, joinHello)

console.log(fns('frankie','lv') // HELLO FREANKIE LV
```

##### 要素

- compose是一个函数，返回一个函数
- 除了第一个函数接收参数，其他的函数接收的函数都是上一个函数的返回值，所以只有第一个函数参数是多参数，而后面的函数都是单参数的
- compose可以接收无限个参数，执行方向是从右到左，初始函数在最右

##### 实现

```
const compose = function(...args) {
    let lens = args.length;
    let count = lens - 1    //记录是否参数执行完毕的退出条件
    let result;     //记录返回值
    return function fn1(...args1) {
        result = args[count].apply(this, args1)
        if (count <= 0) {
            count = lens -1
            return result
        } else {
            count --
            return fn1.call(null, result)
        }
    }
}
```
