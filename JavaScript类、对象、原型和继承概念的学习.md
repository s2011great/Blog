# JavaScript中类、对象、原型和继承的学习笔记
1. 首先我们先来了解什么是面向对象？面向对象编程语言有以下3个特点：

    + 一切事物皆对象
    + 对象具有封装和继承的特性
    + 对象与对象之间使用消息通信，各自存在消息隐藏

    引用自知乎问题“如何用一句话说明什么是面向对象思想？”的高票回答：

    >把一组数据结构和处理它们的方法组成**对象(object)**，把相同行为的对象归纳为**类(class)**，通过类的**封装(encapsulation)**隐藏内部细节，通过**继承(inheritance)**实现类的**特化(specialization)/泛化(generalization)**，通过**多态(polymorphism)**实现基于对象类型的**动态分派(dynamic dispatch)**。

    区别基于类的面向对象、基于原型的面向对象

    JavaScript是通过原型的方式实现面向对象编程。
------------------------------------------------------------
2. 什么是类

    类是一个模板，它描述一类对象的行为和状态。类是一个抽象的概念，对象是类的实体化。
--------------------------------------------------------------
3. JavaScript中定义类和对象的方法

    + 直接量模式
        ```
        var obj = new Object;
        obj.name = "sun";
        obj.showName = function(){  
            alert(this.name);
        };
        ```
        存在问题：如果需要构建多个类似的对象，则还需要重复定义。
    + 工厂模式（抽象了创建具体对象的过程）
        ```
        function createPerson(name, age, job){
            var o = new Object();
            o.name = name;
            o.age = age;
            o.job = job;
            o.sayName = function(){
                alert(this.name);
            };
            return o;
        }
        var person1 = createPerson("Nicholas", 29, "Software Engineer");
        var person2 = createPerson("Greg", 27, "Doctor");
        ```
        存在问题：虽然解决了创建多个相似对象的问题，但并没有解决对象识别的问题（怎样知道一个对象的类型）。
    + 构造函数模式（解决工厂模式不能解决对象类型的问题）
        ```
        function Person(name, age, job){
            this.name = name;
            this.age = age;
            this.job = job;
            this.sayName = function(){
                alert(this.name);
            };
        }

        var person1 = new Person("Nicholas", 29, "Software Engineer");
        var person2 = new Person("Greg", 27, "Doctor");

        alert(person1 instanceof Object);  //true
        alert(person1 instanceof Person);  //true
        ```
        存在问题：构造函数中的方法在每个实例上都会创建一遍，例子中的sayName()在两个对象上完成的功能是一样的，但却是两个函数，有些浪费。
    + 原型模式
        ```
        function Person() {
        }

        Person.prototype.name = "Nicholas";
        Person.prototype.age = 29;
        Person.prototype.job = "Software Engineer";
        Person.prototype.sayName = function() {
            alert(this.name);
        };

        var person1 = new Person();
        person1.sayName();  //"Nicholas"

        var person2 = new Person();
        person2.sayName();  //"Nicholas"

        alert(person1.sayName == person2.sayName);  //true
        ```
        存在问题：

        1. 省略了为构造函数传递初始化参数这一环节，结果就是所有实例在默认情况下都将取得相同的属性值。
        2. 最大问题在于共享的本性，所有的属性和方法对于个体实例来说都是共享的。
        
            这种共享对于函数来说是合适的，对于基本值的属相来说也还可以，各自对象可以在实例上添加同名属性来屏蔽原型属性。
            
            但是对于包含引用类型值的属性来说，问题比较突出。例如：
            ```
            function Person() {
            }

            Person.prototype = {
                construstor: Person,
                name: "Nicholas",
                age: 29,
                job: "Software Engineer",
                friends: ["Shelby", "Court"],
                sayName: function() {
                    alert(this.name);
                }
            };

            var person1 = new Person();
            var person2 = new Person();

            person1.friends.push("Van");

            alert(person1.friends);  //"Shelby, Court, Van"
            alert(person2.friends);  //"Shelby, Court, Van"

            alert(person1.friends == person2.friends);  //true
            ```
            例子中的friends属性的值为引用类型，则在person1实例中改变，也会在person2中反映出来。
    + 组合使用构造函数模式和原型模式
        ```
        function Person(name, age, job) {
            this.name = name;
            this.age = age;
            this.job = job;
            this.friends = ["Shelby", "Court"];
        }

        Person.prototype = {
            constructor: Person,
            sayName: function(){
                alert(this.name);
            }
        };

        var person1 = new Person("Nicholas", 29, "Software Engineer");
        var person2 = new Person("Greg", 27, "Doctor");

        person1.friends.push("Van");
        alert(person1.friends);  //"Shelby, Court, Van"
        alert(person2.friends);  //"Shelby, Court"
        alert(person1.friends === person2.friends)  //false
        alert(person1.sayName === person2.sayName);  //true
        ```
4. 继承

    说完了类和对象，接下来就是类的继承了。
    首先，继承的定义是什么：

    举个例子，A和B都是类，A继承自B。则A为B的子类，B为A的父类或者超类。
    
    所以，继承是可以使得子类具有父类别的各种属性和方法，而不再需要编写相同的代码。子类还可改写这些属性和方法，也可增加新的属性和方法。

    继承的优点：
    
    + 代码的可复用性。
    + 子类可对父类的属性和方法屏蔽，并很容易扩展自身的属性和方法。

    其缺点：
    + 父类和子类高度耦合，子类缺乏独立性。如果父类修改则子类也需修改，可谓一处改处处改。
    + 父类内部细节对子类是可见的。
5. 基于原型的继承

    JavaScript中有各种各样的基于prototype的模拟类继承的实现方式，都是模拟的其他语言的类继承。

    原型的定义：原型是代表事物表象之下的联系。它描述的是事物与事物之间的相似性。

    所以，原型可以通过描述两事物之间的相似关系来复用代码，我们可以把这种复用代码的模式称为原型继承。

    原型继承灵活、初级，只是将相似属性抽取出来作为一个类，然后其它的子类继承父类的属性和方法。
6. JavaScript中实现继承的几种模式

    + 原型链实现继承

        图解
        
        <img src="prototypeChart.png">

        将 B的原型对象 = a; 则 B的实例对象b指向，B的原型对象和A的原型对象、实例对象。

        代码
        ```
        function A() {
            this.name = 'mike';
        }
        A.prototype = {
            showName: function(){
                alert(this.name);
            }
        }
        function B(){}
        B.prototype = new A();

        var b1 = new B();
        b1.name  //'mike'
        b1.showName()  //弹出'mike'
        ```
        b1继承了A的属性和A的原型对象的方法。

        存在问题：
        
            1、子类型无法向父类型传递参数。
            2、属性如果为引用类型，则一个对象实例改变此属性，所有的对象实例此属性都改变。
    + 借用构造函数实现继承

        为了解决原型中包含引用类型的值带来的问题，开发人员使用**借用构造函数**的技术实现继承。
        ```
        function SuperType(){
            this.colors = ["red", "blue", "green"];
        }
        function SubType(){
            //继承了SuperType()
            SuperType.call(this);
        }
        
        var instance1 = new SubType();
        instance1.colors.push("black");
        instance1.colors  //"red, blue, green, black"

        var instance2 = new SubType();
        instance2.colors  //"red, blue, green"
        ```
        存在问题：

            每一次创建实例对象，都会创建一次函数，导致函数无法复用。
    + 组合继承

        将原型链和借用构造函数的技术组合到一块。
        ```
        function SuperType(name){
            this.name = name;
            this.colors = ["red", "blue", "green"];
        }
        SuperType.prototype.sayName = function(){
            alert(this.name);
        };

        function SubType(name, age){
            SuperType.call(this, name);
            this.age = age;
        }

        SubType.prototype = new SuperType();
        SubType.prototype.constructor = SubType;
        SubType.prototype.sayAge = function(){
            alert(this.age);
        };

        var instance1 = new SubType("Nicholas", 29);
        instance1.colors.push("black");
        instance1.colors;  //"red, blue, green, black"
        instance1.sayName();  //"Nicholas"
        instance1.sayAge();  //29

        var instance2 = new SubType("Greg", 27);
        instance2.colors;  //"red, blue, green"
        instance2.sayName();  //"Greg"
        instance2.sayAge();  //27
        ```
    + 原型式继承

        Object.create()  接受两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。
        ```
        var person = {
            name: "Nicholas",
            friends: ["Shelby", "Court", "Van"]
        };

        var anotherPerson = Object.create(person);
        anotherPerson.name = "Greg";
        anotherPerson.friends.push("Rob");

        var yetAnotherPerson = Object.create(person);
        yetAnotherPerson.name = "Linda";
        yetAnotherPerson.friends.push("Barbie");

        alert(person.friends);  //"Shelby, Court, Van, Rob, Barbie"
        ```
    + 寄生式继承

        ```
        function object(o){
            function F(){}
            F.prototype = o;
            return new F();
        }
        function createAnother(original){
            var clone = object(original);
            clone.sayHi = function(){
                alert("Hi");
            };
            return clone;
        }

        var person = {
            name: "Nicholas",
            friends: ["Shelby", "Court", "Van"]
        };
        var anotherPerson = createAnother(person);
        anotherPerson.sayHi();  //"hi"
        ```
    + 寄生组合式继承

        ```
        function inheritPrototype(subType, superType){
            var prototype = Object(superType.prototype);
            prototype.constructor = subType;
            subType.prototype = prototype;
        }

        function SuperType(name){
            this.name = name;
            this.colors = ["red", "blue", "green"];
        }
        SuperType.prototype.sayName = function(){
            alert(this.name);
        };
        function SubType(name, age){
            SuperType.call(this, name);
            this.age = age;
        }

        inheritPrototype(SubType, SuperType);

        SubType.prototype.sayAge = function(){
            alert(this.age);
        };
        ```
7. ES6中类的实现

    基本类声明语法
    ```
    class PersonClass {
        //等价于构造函数
        constructor(name){
            this.name = name;
        }
        //等价于为原型对象添加方法
        sayName(){
            console.log(this.name);
        }
    }

    let person = new PersonClass("Nicholas");
    person.sayName();  //"Nicholas"
    ```

