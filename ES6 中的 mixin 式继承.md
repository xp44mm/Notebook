## ES6 中的 mixin 式继承

在 ES6 中，我们可以采用全新的基于类继承的 *mixin* 模式设计更优雅的“语义化”接口，这是因为 ES6 中的 `extends` 可以继承动态构造的类，类就是函数。在说明它的好处之前，我们先看一下 ES6 中的 `Serializable`实现：

```javascript
const Serializable = (Sup=Object) => class extends Sup {
    constructor(...args) {
        super(...args)
        
        if (typeof this.constructor.stringify != "function") {
            throw new ReferenceError("Please define stringify method to the Class!")
        }
        if (typeof this.constructor.parse != "function") {
            throw new ReferenceError("Please define parse method to the Class!")
        }
    }

    toString() {
        return this.constructor.stringify(this)
    }
}
```

在上面的代码里，我们改变了 `Serializable`，让它成为一个动态返回类的函数。注意`this.constructor`从实例动态访问静态函数。

```javascript
class Person {
    constructor(name, age, gender) {
        Object.assign(this, { name, age, gender });
    }
}

class Employee extends Serializable(Person) {
    constructor(name, age, gender, level, salary) {
        super(name, age, gender)
        this.level = level
        this.salary = salary
    }
    static stringify(employee) {
        let { name, age, gender, level, salary } = employee
        return JSON.stringify({ name, age, gender, level, salary })
    }
    static parse(str) {
        let { name, age, gender, level, salary } = JSON.parse(str)
        return new Employee(name, age, gender, level, salary)
    }
}

```

然后我们通过 `class Employee extends Serializable(Person)` 来实现可序列化，在这里我们没有可序列化 `Person` 本身，而将 `Serializable` 在**语义上**变成一种修饰，即 **`Employee` 是一种可序列化的 `Person`**。于是，我们要 `new Person` 就不会报错了： 

```javascript
test('test mixin Person', () => {
    let person = new Person("john", 22, "m")

    expect(person).toBeInstanceOf(Person)
    expect(person.name).toBe("john")
    expect(person.age).toBe(22)
    expect(person.gender).toBe("m")
})
test('test mixin Employee', () => {
    let employee = new Employee("jane", 25, "f", 1, 1000)
    let employee2 = Employee.parse(employee.toString())

    expect(employee2).toEqual(employee)
    expect(employee2).toBeInstanceOf(Employee)
    expect(employee2).toBeInstanceOf(Person)
    expect(employee2).not.toBe(employee)
})
```



利用 *mixin* 模式，我们还可以实现对原生类的继承，例如： 

```javascript
class MySet extends Serializable(Set) {
    static stringify(s) {
        return JSON.stringify([...s]);
    }
    static parse(data) {
        return new MySet(JSON.parse(data));
    }
}

test('test mixin MySet', () => {
    let s1 = new MySet([1, 2, 3, 4])
    let s2 = MySet.parse(s1.toString())

    expect(s2).toEqual(s1)
    expect(s2).not.toBe(s1)
})
```

通过 `MySet` 继承 `Serializable(Set)`，我们得到了一个可序列化的 `Set` 类。



我们还可以定义其他的“修饰符”，然后将它们组合使用，比如： 

```javascript
const Immutable = Sup => class extends Sup {
    constructor(...args) {
        super(...args)
        Object.freeze(this)
    }
}

class MyArray extends Immutable(Serializable(Array)) {
    static stringify(arr) {
        return JSON.stringify({ Immutable: arr })
    }
    static parse(data) {
        return new MyArray(...JSON.parse(data).Immutable)
    }
}

test('test mixin Immutable', () => {
    let arr1 = new MyArray(1, 2, 3, 4)
    let arr2 = MyArray.parse(arr1.toString())

    expect(() => arr1.push(5)).toThrow(TypeError)
    expect(arr1).toEqual(arr2)
    expect(arr1).not.toBe(arr2)
})
```

上面的例子里，我们通过 `Immutable` 修饰符定义了一个不可变数组，同时通过 `Serializable` 修饰符修改了它的序列化存储方式，而这一切，通过定义 `class MyArray extends Immutable(Serializable(Array))` 来实现。

### 实现多重继承

实际上仍然是一个一个的继承链。



## 总结

我们看到了 ES6 的 **mixin** 式继承的优雅和灵活，相信大家对它强大的功能和非常漂亮的装饰器语义有了比较深刻的印象。