## TypeScript

### 数据类型

#### 布尔类型(boolean)

#### 数字类型(number)

#### 字符串类型(string)

#### 数组类型(array)

#### 元组类型(tuple)
> 表示已知数量和类型的数组

|元组|数组|
|:-|-:|
|每一项可以是不同的类型|每一项都是同一种类型|
|有预定义的长度|没有长度限制|
|用于表示一个固定的结构|用于表示一个列表|

#### 枚举类型(enum)

##### 普通枚举

##### 常数枚举
> 常数枚举会在编译阶段被删除，并且不能包含计算成员

#### 任意类型(any)
> 第三方库没有提供类型文件时常用any，数据结构太复杂难以定义时也常用any

#### null和undefined
> null为空 undefined为未定义；它们都是其他类型的子类型

#### void类型
> void表示没有任何类型

#### never类型
> null、undefined的子类型，代表不会出现的值，函数返回never，函数永远不会停止，函数会报错

#### 类型推论
> 定义时未赋值会推论成any类型

#### 包装对象(wrapper object)
> java中的装箱和拆箱

#### 联合类型(union types)
> 表示取值可以为多种类型中的一种，未赋值时联合类型只能访问两个类型共有的属性和方法

#### 类型断言
> 类型断言可以将一个联合类型变量指定为一个更加具体的类型，但不能将联合类型断言为不存在的类型；'!'为非空断言

#### 字面量类型
> 可以把字符串、数字、布尔值字面量组成一个联合类型

#### 字符串字面量 vs 联合类型
> 字符串字面量只能约束是几个字符串中的一个，联合类型可以取值到多种类型的一种类型

### 函数

#### 函数的定义
> 函数定义，指定参数的类型和返回值的类型
```
    function 函数名(形参名:形参类型):返回值类型{
        ...
    }
```

#### 函数表达式
> 定义函数类型
```
    type 函数名 = (形参1:形参类型1,形参2:形参类型2)=>返回值类型;
```

#### 没有返回值
> void

#### 可选参数
> ts中形参和实参必须一样，不一样就啊哟配置可选参数，而且必须是最后一个参数
```
    function 函数名(形参1:参数类型1,可选参数?:可选参数类型)){
        ...
    }
```

#### 默认参数
> 形参名:形参类型=默认值

#### 剩余参数
> ...形参名:形参类型[]

#### 函数重载
> 不用于java中的重载，TypeScript中重载表示同一个函数提供多个函数类型定义

### 类

#### 如何定义类
- "strictPropertyInitialization":true /启用类属性初始化的严格检查/
- name!:string

#### 存取器
- 通过存取器来改变一个类中属性的读取和赋值
- 构造函数
    - 初始化类的成员变量属性
    - 类的对象创建时自动调用执行
    - 没有返回值

#### 参数属性

#### readonly
- 只能修饰类初始化中的变量
- 在TypeScript中，const是常量标志符，其值不能被重新分配
- 允许将interface、type、class上的属性标识为readonly
- readonly只是在编译阶段进行代码检查，const会在运行时检查

#### 继承

#### 类里面的修饰符

#### 静态属性 静态方法

#### 装饰器
- 装饰器是一种特殊类型的声明，它能够被附加到类声明、方法、属性或参数上，可以修改类的行为
- 常见装饰器
    - 类装饰器
    - 属性装饰器
    - 方法装饰器
    - 参数装饰器
- 装饰器的写法分为普通装饰器和装饰器工厂
- 执行顺序：
    - 属性装饰器和方法装饰器先执行，谁在前面谁先执行
    - 参数装饰器属于方法装饰器的一部分，参数会紧挨着方法执行
    - 类装饰器最后执行
    - 同类型装饰器就近原则，谁靠的越近先修饰

##### 类装饰器
- 类装饰器在类之前声明，用来监视、修改或替换类定义
> 普通装饰器
```
    interface 类名{
        属性1:属性类型1;
        属性2:属性类型2;
    }
    @装饰器名
    class 类名{
        constructor(){

        }
    }
```
> 装饰器工厂
```
    interface 类名{
        属性1:属性值1;
        属性2:属性值2;
    }
    function 装饰器名(属性名:属性类型){
        return function 装饰器名(目标名:any){
            目标名.prototype.属性1=属性值1;
            目标名.prototype.属性2=function(){
                ...
            }
        }
    }
    @装饰器名(属性值)
    class 类名{
        constructor(){

        }
    }
```

##### 属性装饰器
- 属性装饰器表达式会在运行时当作函数被调用，传入两个参数
    - 第一个参数对于静态成员是类的构造函数，对于实例成员是类的原型对象
    - 第二个参数是属性的名称

##### 方法装饰器
- 方法装饰器用来装饰方法
    - 第一个参数对于静态成员是类的构造函数，对于实例成员是类的原型对象
    - 第二个参数是方法的名称
    - 第三个参数是方法描述符

##### 参数装饰器
- 会在运行时当作函数被调用，可以使用参数装饰器为类的原型增加一些元数据
    - 第一个参数对于静态成员是类的构造函数，对于实例成员是类的原型对象
    - 第二个参数是方法的名称
    - 第三个参是在函数列表中索引

#### 抽象类
- 抽象描述一种抽象的概念，无法被实例化，只能被继承
- 无法创建抽象类的实例
- 抽象方法不能再抽象类中实现，只能在抽象类的具体子类中实现，而且必须实现

> 关键字对比
|名称|访问修饰符|
|:-|-:|
|访问控制修饰符|private protected public|
|只读属性|readonly|
|静态属性|static|
|抽象类、抽象方法|abstract|

#### 抽象类 vs 接口
- 接口：不同类之间公有的属性或方法；抽象类：其他类继承的基类，抽象类不允许被实例化
- 接口仅用于描述，既不提供方法的实现，也不为属性进行初始化；抽象类是一个无法被实例化的类，可以实现属性和方法的初始化
- 一个类只能继承一个类或抽象类，但可以实现多个接口
- 抽象类也可以实现接口

#### 抽象方法
- 抽象类和方法不包含具体实现，必须在子类中实现
- 抽象方法只能出现在抽象类中
- 子类可以对抽象类进行不同的实现

#### 重写(override) vs 重载(overload)
- 重写是指子类重现继承自父类中的方法
- 重载是指为同一个函数提供多个类型定义

#### 继承 vs 多态
- 继承(inheritance)：子类继承父类，子类除了拥有父类所有特性外，还有一些更具体的特性
- 多态(polymorphism)：由继承而产生了相关的不同的类，对同一个方法可以有不同的行为

> class中的super，有两种指向，在静态方法和构造函数中，指向父类；在普通函数中，指向父类的prototype

### 接口

#### 接口
> interface中可以用分号或者逗号分割每一项，也可以什么都不加

##### 对象的形状

##### 行为的抽象

##### 任意属性

#### 接口的继承
> 一个接口可以继承自另外一个接口

#### readonly
> 用readonly定义只读属性可以避免由于多人协作或者项目较为复杂等原因而造成对象的值被重写

#### 函数类型接口
> 对方法传入的参数和返回值进行约束

#### 可索引接口
> 对数组和对象进行约束

#### 类接口
> 对类的约束

#### 构造函数的类型
> TypeScript中可以用interface来描述类，也可以使用其中特殊的new()关键字来描述类的构造函数类型

### 泛型
> 泛型(Generics)：在定义函数、接口或类的时候，不预先指定具体的类型，而是在使用的时候再指定类型的一种特性。泛型T作用域只限于函数内部使用

#### 泛型函数

#### 类数组

#### 泛型类

#### 泛型接口

#### 多个类型参数

#### 默认泛型类型
```
    <T=默认类型>
```

#### 泛型约束

#### 泛型类型别名
> type 可以表达更加复杂的类型

#### 泛型接口 vs 泛型类型别名
> 接口是创建了一个新的名字，在其他任意地方背调偶用，而类型别名并不创建新的名字，报错的时候不会使用别名；类型别名不能被extends和implements，类型别名更适合使用联合类型或元组类型

### 结构类型系统

#### 接口的兼容性
> 如果传入的变量和声明的乐行不匹配，ts就会进行兼容性检查，只要目标类型中声明的属性变量在源类型中都存在就是兼容的

#### 基本类型的兼容性

#### 类的兼容性
> TypeScript是结构类型系统，只会对比结构而不在意类型

#### 函数的兼容性
> 先比较函数的参数，再比较函数的返回值

#### 函数参数的协变
> 目标如果能够兼容源就可以

#### 泛型的兼容性
> 先判断具体的类型再进行兼容性判断

#### 枚举的兼容性
> 枚举类型与数字类型兼容，数字类型与枚举类型兼容；不同枚举类型之间不兼容

### 类型保护
> 类型保护是通过关键字判断出分支的类型，在编译的时候能够通过类型信息确定某个作用域内变量的类型

#### typeof 类型保护

#### instanceof类型保护

#### null保护

#### 链判断运算符
> ?.

#### 可辨识的联合类型

#### in 操作符

#### 自定义的类型保护
