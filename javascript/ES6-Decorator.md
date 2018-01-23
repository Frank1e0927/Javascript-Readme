## ES Decorators



#### 1.什么是Decorators
> 意在装饰修改类或者函数的行为

**看一段代码**

```
@addVisiable
class MyClass {
    // ...
}

function addVisiable(target) {
    target.visiable = true;
}

MyClass.visiable    // true
```
1：以上代码，addVisiable对MyClass这个类进行了修改，给MyClass增加了一个静态属性visiable。addVisiable这个函数的参数target就是这个类本身

2：修饰器是一个对类进行处理的函数。修饰器函数的第一个参数，就是所要修饰的目标类。

**假如一个参数不够用**

可以再修饰器外层再封装一层函数来传递参数

```
function addVisiable(visiable) {
    return function(target) {
        target.visiable = visiable
    }
}

@addVisiable(true)
class MyClass{}

MyClass.visiable    // true
```

**以上都是添加静态属性，如何添加一个实例属性**

可以通过*target.prototype*来操作


```
function addVisiable(target) {
    target.prototype.visiable = true
}

@addVisiable
class MyClass{}

const obj = new MyClass()
obj.visiable    // true
```
**把多个需要实例化的属性添加到clas中**


```
  function mixins(...list) {
    return function(target) {
      Object.assign(target.prototype, ...list)
    }
  }

  const Foo = {
    foo() {
      console.log('foo')
    },
    game() {
      console.log('game')
    }
  }
  
  @mixins(Foo)
  class MyClass {

  }
  const obj = new MyClass();
  obj.game()    // 'game'
  obj.foo()     // 'foo'
```

#### 2.方法的修饰
> 修饰器不仅仅可以修饰类，同样可以修饰类的属性。

```
function readonly(target, name, descriptor) {
    /* 原discriptor对象属性如下 */
    /* 
        {
            value: specifiedFunction,
            enumerable: false,  // 枚举
            configurable: true,
            writable: true,
        }
    */
    descriptor.wirtable = false;
    return descriptor
}

readonly(Person.prototype, 'name', descriptor)
等同于
Object.definedProperty(Person.prototype, 'name', descriptor)

class Game {
    @readonly
    name() {
        return `${this.first} ${this.last}`
    }
}
```
**下例是@log修饰器，起到输出日志作用**


```
  function log(target, name, descriptor) {
    console.log('descriptor', descriptor)
    console.log('target constructor', target.constructor)
    console.log('name', name)
    var oldValue = descriptor.value;

    descriptor.value = function() {
      console.log(`Calling ${name} with`, arguments)
      return oldValue.apply(null, arguments);
    }

    return descriptor
  }
  class Math {
    @log
    add(a, b) {
      return a + b;
    }

    @log
    game(){
      console.log('game')
    }
  }
  const math = new Math()
  math.add(1,3)
```

以上的log三个参数，暴露出几个知识点
- descriptor是修饰对象的各种属性
- target的构造函数一定是这个类本身
- name也就是修饰的函数的函数名

**多重修饰器会从内向外开始**


```
class Test {
    @add(1)
    @add(2)
    method() {}
}
```

