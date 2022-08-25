[具体介绍请点击此处](http://xihale.top/%E6%9C%80%E6%96%B0%E6%97%A0%E5%A4%87%E6%A1%88%E7%BD%91%E7%AB%99%E6%90%AD%E5%BB%BA%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)

> 请自行修改一下代码以达到需求

## 子页代码(simplely)

```html
<script>
    top.postMessage(location.pathname+location.search+location.hash, "*");
</script>
```

## cookie 跨站问题 (登录问题)

> 需要https,请自行注册https(ip)

```
Header always edit Set-Cookie ^(.*)$ "$1;Secure;SameSite=None;"
```

## 子页代码

```html
<script>
class Dep {                  // 订阅池
    constructor(name){
        this.id = new Date() //这里简单的运用时间戳做订阅池的ID
        this.subs = []       //该事件下被订阅对象的集合
    }
    defined(){              // 添加订阅者
        Dep.watch.add(this);
    }
    notify() {              //通知订阅者有变化
        this.subs.forEach((e, i) => {
            if(typeof e.update === 'function'){
                try {
                   e.update.apply(e)  //触发订阅者更新函数
                } catch(err){
                    console.warr(err)
                }
            }
        })
    }
    
}
Dep.watch = null;

class Watch {
    constructor(name, fn){
        this.name = name;       //订阅消息的名称
        this.id = new Date();   //这里简单的运用时间戳做订阅者的ID
        this.callBack = fn;     //订阅消息发送改变时->订阅者执行的回调函数     
    }
    add(dep) {                  //将订阅者放入dep订阅池
       dep.subs.push(this);
    }
    update() {                  //将订阅者更新方法
        var cb = this.callBack; //赋值为了不改变函数内调用的this
        cb(this.name);          
    }
}

var addHistoryMethod = (function(){
        var historyDep = new Dep();
        return function(name) {
            if(name === 'historychange'){
                return function(name, fn){
                    var event = new Watch(name, fn)
                    Dep.watch = event;
                    historyDep.defined();
                    Dep.watch = null;       //置空供下一个订阅者使用
                }
            } else if(name === 'pushState' || name === 'replaceState') {
                var method = history[name];
                return function(){
                    method.apply(history, arguments);
                    historyDep.notify();
                }
            }
            
        }
}())

window.addHistoryListener = addHistoryMethod('historychange');
history.pushState =  addHistoryMethod('pushState');
history.replaceState =  addHistoryMethod('replaceState');

window.addHistoryListener('history',function(){
    top.postMessage("history "+location.pathname+location.search+location.hash, "*");
});

const title_change= new MutationObserver(function(mutations) {
    top.postMessage("title "+mutations[0].target.innerText, "*");
})

title_change.observe(document.querySelector('title'), { characterData: true, subtree: true, childList: true })

top.postMessage("title "+document.title, "*");

console.log(document.title);
</script>
```