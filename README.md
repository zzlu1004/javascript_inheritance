# javascript继承

## 借助构造函数实现继承
### 举例：不推荐使用
```javascript
  function Parent1() {
    this.name = 'parent1'
  }

  Parent1.prototype.say = function () {}

  function Child1() {
    // 执行Parent1.call(this)，相当于在Child1()函数中执行this.name = 'parent1'
    // 所以，new Child1()后，在实例child1中会有name属性
    Parent1.call(this)
    this.type = 'child1'
  }
  
  var child1 = new Child1()
  
  console.log(child1) // Child1 {name: "parent1", type: "child1"}
  
  // 执行如下语句会报错：(intermediate value).say is not a function
  // Parent1.call(this)只会继承实例属性（如：name = 'parent1'），并不会继承原型上的方法（如：say=function(){}）
  console.log(child1.say()) 
```
### 说明
- 只继承实例属性
- 没有继承原型属性

## 借助原型链实现继承
### 举例：不推荐使用
```javascript
  function Parent2() {
    this.name = 'parent2'
    this.play = [1, 2, 3]
  }

  function Child2() {
    this.type = 'child2'
  }

 // Child2的实例可以继承Parent2的原型属性
 // new Parent2()执行后，创建的对象，有name = 'parent2'，play = [1, 2, 3]
 // 又play属性，是引用，所以执行 s1.play.push(4)后，s1.play与s2.play都会改变
  Child2.prototype = new Parent2()

  var s1 = new Child2()
  var s2 = new Child2()
  console.log(s1.play, s2.play)
  s1.play.push(4)
```
### 说明
- 同时继承了实例属性和原型属性
- Parent2的实例属性this.play = [1, 2, 3]是引用，所以Child2的所有实例共享


## 组合方式
### 举例：不推荐使用
```javascript
  function Parent3() {
    this.name = 'parent3'
    this.play = [1, 2, 3]
  }

  function Child3() {
    // 执行Parent3.call(this)后，Child3的实例属性上会有this.play = [1, 2, 3]
    Parent3.call(this)
    this.type = 'child3'
  }

  // 执行Child3.prototype = new Parent3()后，
  // Child3.prototype也有this.play = [1, 2, 3]，即this.play会初始化两次
  // s3.play与s4.play，访问的是实例属性play，即通过Parent3.call(this)创建的play
  Child3.prototype = new Parent3()
  
  var s3 = new Child3()
  var s4 = new Child3()
  s3.play.push(4)
  console.log(s3.play, s4.play)
```


## 组合继承的优化1
### 举例：不推荐使用
```javascript
 function Parent4() {
    this.name = 'parent4'
    this.play = [1, 2, 3]
  }

  function Child4() {
    Parent4.call(this)
    this.type = 'child4'
  }
  
  // 执行Child4.prototype = Parent4.prototype后，
  // Child4实例上只有一份实例属性，符合预期
  // 但，Child4.prototype.constructor === Parent4，不符合预期
  // 期望Child4.prototype.constructor === Child4
  Child4.prototype = Parent4.prototype
  var s5 = new Child4()
  var s6 = new Child4()
  console.log(s5, s6)

  console.log(s5 instanceof Child4, s5 instanceof Parent4)
  console.log(s5.constructor)
```

## 组合继承的优化2
### 举例：推荐
```javascript
 function Parent5() {
    this.name = 'parent5'
    this.play = [1, 2, 3]
  }

  function Child5() {
    // 执行Parent5.call(this)后，Child5实例只有一份属性，符合预期
    Parent5.call(this)
    this.type = 'child5'
  }

  // Object.create(Parent5.prototype)返回一个以Parent5.prototype为原型的空对象
  // 让Child5.prototype指向创建的空对象，
  Child5.prototype = Object.create(Parent5.prototype)
  // 给空对象添加constructor属性，不会污染Parent5.prototype，符合预期
  Child5.prototype.constructor = Child5
```
### 说明
- 当前这种方式，是javascript实现继承的完美方式
- 各大框架，如VUE中，也是用这种方式实现的继承，组件构造函数继承VUE的构造函数


## 模拟对象创建的过程
```javascript
  var new2 = function (func) {
    var o = Object.create(func.prototype)
    var k = func.call(o)
    if (typeof k == 'Object') {
      return k
    }
    return o
  }

  function M() {
    this.name = '请叫我斗图王'
  }

  var m = new2(M)
  console.log(m, m.__proto__ == M.prototype) // {name: "请叫我斗图王"}name: "请叫我斗图王"__proto__: Object true
```

