### 读写文件，监听文件变化

> 相同目录下创建文件夹，done和watch，启动以下程序，如果对watch文件发生了变化，就会输出到done文件下

```
const events = require('events');
const util = require('util');
const fs = require('fs');

let watchDir = './watch'
let processedDir = './done'

class Watcher extends events.EventEmitter {
    constructor(watchDir, processedDir) {
        super()
        this.watchDir = watchDir;
        this.processedDir = processedDir;
    }
    watch() {
        console.log('get watch')
        const watcher = this;
        fs.readdir(this.watchDir, function(err, files) {
            if (err) throw err
            for(var index in files) {
                watcher.emit('process', files[index])
            }
        })
    }
    start() {
        const watcher = this
        //  监听文件变化的api，详情查看nodejs官网
        fs.watchFile(this.watchDir, function() {
            watcher.watch()
        })
    }
}

const watcher = new Watcher(watchDir, processedDir);

watcher.on('process', function process(file){
    const watchFile = this.watchDir + '/' + file;
    const processedFile = this.processedDir + '/' + file.toLowerCase();

    fs.rename(watchFile, processedFile, function(err) {
        if (err) throw err;
    })
})

watcher.start();
console.log('statr')
```

### connect中间件
> npm install connect
类似于express，express某些机制也是connect的上层概念


#### 单一中间件

```
const connect = require('connect')
const app = connect()

function logger(req, res, next) {
    console.log(req.method, res.url)
}
app.use(logger)
app.listen(3333)
```
如果不相应res.end，则会直接返回404到客户端当中。

### 多个中间件


```
const connect = require('connect')
const app = connect()

function logger(req, res, next) {
    console.log(req.method, res.url)
}

function hello(req, res) {
    res.setHeader('Content-Type', 'text/plain')
    res.end('hello world')
}
app
    .use(logger)
    .use(hello)
app.listen(3333)
```
加上了res.end，则会正常返回，并且在客户端显示hello world，connect可以链式调用
