title: "面向对象js"
date: 2012-07-04 00:00:00
category: [前端]
tags: [js]
---
一些很久前写的东西

##面向对象的好处
代码以类为单元进行管理，把相关的数据和方法封装到类中。


##js对象的本质
js的对象实质就是java的map， 它是key和value对的集合
```javascript
var estate = {};
estate.estateName = '达安花园';
estate.address = '静安区...';
```

访问对象属性，通过“.”或“[]”进行访问
```javascript
estate.estateName 或 estate.['estateName']
```

遍历
```javascript
for(var propName in estate){
    alert(propName+'='+estate[propName]);
}
```


##如何在js中写类（constructor/prototype pattern）
```javascript
//实例属性，实例化的每个对象都会有自己的一份拷贝
function Person(name, age){               //构造函数：用来构造对象
     this.name = name;
     this.age = age;
}

//原型（共享）属性， 所有的对象共享一份
Person.prototype = {
     constructor: Person,
     toString : function(){
          return "name="+this.name+',age='+this.age;
     }
};

//当执行new操作的时候， 构造函数Person中的this即指向变量person，通过构造函数的语句，即给person变量添加了name和age属性
var person = new Person('zhangsan', 25); 
alert(person.toString());
```


##js中实现继承
```javascript
function Student(name, age, score){
     Person.call(this, name, age);               //借用父类的构造函数：目的是通过父类的构造函数， 把父类中定义的属性复制到子类对象上
     this.score = score;
}

Student.prototype = new Person();               //子类的原型赋值为父类的对象： 目的是共享父类的方法
Student.prototype.toString= function(){
     return Person.prototype.show.apply(this)+",score="+this.score+"";
}
         
var student = new Student('zhanglg', 30, 98);
alert(student.toString());
```

当在对象上访问某个属性时， 先从对象上找该属性，找到则返回该属性， 否则向上回溯到原型中去找该属性， 找到则返回， 否则为undefined