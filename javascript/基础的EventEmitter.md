## 实现一个Event Emitter

> 事件驱动 （event-dirven）模型是javascript编程中绕不过的概念，事件发射器(event-emitter)则是事件驱动模型的核心。Event Emitter与Class很类似，几乎稍大一点的前端项目，都会实现一遍，大多数情况下，会被抽象成为一个Emmiter类，通过实例方法，提供事件绑定(on/bind)、事件触发(fire/trigger)等功能

实现一个基本的Event-Emitter，主要逻辑包括
- 维护每个事件对应的回调函数(listener)队列
- 在事件被出发的时候逐个调用


version-1
```
function Events() {

}
var eventProto = Events.prototype
// regular expression used to split event strings
var eventSplitter = /\s+/

eventProto.on = function(events, handler) {

  events = events.split(eventSplitter)
  
  this._callbacks = this._callbacks || {}
  this._callbacks[events] = this._callbacks[events] || []
  this._callbacks[events].push(handler)

  return this;
}

eventProto.off = function(events, handler) {
  var list = this._callbacks && this._callbacks[type]

  if (list) {
    var i = list.length;
    while(i--) {
      if (list[i] === handler) {
        list.splice(i, 1)
      }
    }
  }

  return this;
}

eventProto.once = function(events, handler) {
  var self = this;

  function wrapper() {
    handler.apply(self, arguments);
    self.off(type, wrap)
  }
  this.on(type, wrapper)
  return this
}

eventProto.emit = function(events, data) {
  var callbacks = this._callbacks && this._callbacks[events];
  if (callbacks) {
    callbacks.forEach(cb => {
      cb(data)
    })
  }
}

var test1 = new Events()

test1.on('test', () => {
  console.log('test on event')
})

test1.emit('test')
```
