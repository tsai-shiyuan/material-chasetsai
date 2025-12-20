# CS50P: 8. Object-Oriented Programming

## Object-Oriented Programming

### tuple

A `tuple` is a sequence of values. Unlike a `list`, a `tuple` can’t be modified (does not support item assignment)

Like x, y, z

```python
def main():
    name, house = get_student()
    print(f"{name} from {house}")
def get_student():
    name = input("Name: ")
    house = input("House: ")
    return name, house		# return a turple with 2 elements
if __name__ == "__main__":
    main()
```

tuple & index:

```python
def main():
    student = get_student()
    print(f"{student[0]} from {student[1]}")
def get_student():
    name = input("Name: ")
    house = input("House: ")
    return (name, house)
```

### list

```python
def main():
    student = get_student()
    if student[0] == "Padma":
        student[1] = "Ravenclaw"	#可修改
    print(f"{student[0]} from {student[1]}")
def get_student():
    name = input("Name: ")
    house = input("House: ")
    return [name, house]
```

### dict

```python
def main():
    student = get_student()
    if student["name"] == "Padma":
        student["house"] = "Ravenclaw"
    print(f"{student['name']} from {student['house']}")
def get_student():
    student = {}
    student["name"] = input("Name: ")
    student["house"] = input("House: ")
    return student
```

line 5 的 `'name'` 必须用单引号括，因为 `print(f" ")`

line 7 ~10 :

```python
name = input("Name: ")
house = input("House: ")
return {"name": name, "house": house}
```

## classes

Classes are a way by which, in object-oriented programming, we can **create our own type of data and give them names**.

[python’s documentation of classes](https://docs.python.org/3/tutorial/classes.html)

class is like blueprint of a house, object is like a house

### create new data type

```python
class Student:
    ... # i will come back to implement this later
```

### create objects from classes

```python
def get_student():
    student = Student()	# create object/instance
    student.name = input("Name: ")  # . inside the student
    student.house = input("House: ")
    return student
```

`.` access attributes of this variable `student` of class `Student`

### standardize

lay some groundwork for the attributes that are expected inside an object whose class is `Student`

define a class = get a function? eg. Student可以当作函数

```python
def get_student():
    name = input("Name: ")
    house = input("House: ")
    student = Student(name, house)
    return student
```

### methods

classes come with certain methods/functions inside of them and we can define

#### \__init__

initialize the contents of an object from a class, we define this method

```python
class Student:
    def __init__(self, name, house):
        self.name = name
        self.house = house
def main():
    student = get_student()
    print(f"{student.name} from {student.house}")
def get_student():
    name = input("Name: ")
    house = input("House: ")
    student = Student(name, house)  # construct call, will goto def __init__
    return student
```

`def __init__` : create a function within `class Student`, called a “method”

self gives us access to the current object that was just created

`self.name = name` : add variables to objects like add values to keys in dict

line 11 `student = Student(name, house)` python will call \__init__ for us, 在这一步就已经分配内存

\__init__ 只会执行一次

#### \__str__

a specific function by which you can print the attributes of an object

```python
class Student:
    def __init__(self, name, house):
        if not name:
            raise ValueError("Missing name")    
        if house not in ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"]:
            raise ValueError("Invalid house")
        self.name = name
        self.house = house
    def __str__(self):
        return f"{self.name} from {self.house}"
def main():
    student = get_student()
    print(student)
```

#### our own methods

```python
class Student:
    def __init__(self, name, house, patronus):
        if not name:
            raise ValueError("Missing name")   
        if house not in ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"]:
            raise ValueError("Invalid house")
        self.name = name
        self.house = house  
        self.patronus = patronus
    def __str__(self):
        return f"{self.name} from {self.house}"
    def charm(self):
        match self.patronus:
            case "Stag":
                return "🐴"
            case "Otter":
                return "🦦"
            case "Jack Russell terrier":
                return "🐶"
            case _:
                return "🪄"
def main():
    student = get_student()
    print("Expecto Patronum!")
    print(student.charm())
```

至少含有一个参数 `self` (默认)

## raise

What if something goes wrong? What if someone tries to type in something random? What if someone tries to create a student without a name?

error, validate input

```python
class Student:
    def __init__(self, name, house):
        if not name:
            raise ValueError("Missing name")    # treat ValueError as a function
        if house not in ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"]:
            raise ValueError("Invalid house")
        self.name = name
        self.house = house  # add variables to objects like add values to keys in dict
```

`if not name:` 等价于 `if name == "":`

## Decorators

Functions that modify the behavior of other functions (通常是 @xxx)

### properties

- an attribute that has more defense machanisms into place

- a function

#### Getters & Setters

getter: a function for a class that gets some attributes

setter: a function that sets some value

Require: to access/set an attribute, we go through some function(getter/setter)

```python
class Student:
    def __init__(self, name, house):
        if not name:
            raise ValueError("Missing name")    
        self.name = name
        self.house = house  
    
    def __str__(self):
        return f"{self.name} from {self.house}"		
  
  	# Getter
    @property
    def house(self):
        return self._house
    #Setter
    @house.setter
    def house(self, house):
        if house not in ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"]:
            raise ValueError("Invalid house")
        self._house = house
```

当 python 读到 `student.house = "Number Four, Privet Drive"` 这句话，又有一个setter method ，还有 `=` 表示赋值，python 会自动执行 setter method 

line 6的 `self.house = house` 同样会执行 house 函数

总结：`xx.house =` 就会执行house函数

#### @properties

@property(line 2): Doing so defines `house` as a property of our class. With `house` as a property, we gain the ability to define how some attribute of our class, `_house`, should be set and retrieved; we can now define a function called a “setter”, via `@house.setter`, which will be called whenever the house property is set

修改 line 14 和 line 20 `self._house` ：When a user calls `student.house`, they’re getting the value of `_house` through our `house` “getter”

* _house 是 attribute
* house 是 property

见到 `_house` 我们尽量就不去修改这个变量，尽管可以

#### 优化版本

```python
class Student:
    def __init__(self, name, house):
        self.name = name
        self.house = house  
    
    def __str__(self):
        return f"{self.name} from {self.house}"
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, name):
        if not name:
            raise ValueError("Missing name")   
        self._name = name

    @property
    def house(self):
        return self._house
    
    @house.setter
    def house(self, house):
        if house not in ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"]:
            raise ValueError("Invalid house")
        self._house = house

def main():
    student = get_student()
    print(student)

def get_student():
    name = input("Name: ")
    house = input("House: ")
    return Student(name, house)

if __name__ == "__main__":
    main()
```

## Previous Work

### int

`int` 是一个 class，a blueprint for creating objects of type [int](https://docs.python.org/3/library/functions.html#int)

```python
class int(x, base=10)
```

### str

`str` 也是一个 class，使用 `str.lower()` ，就在从 str 中取 object，通过 method lower  让它成为小写

```python
class str(object='')
```

### list

```python
class list([iterable])
```

### type

```python
print(type(50))
```

返回 `<class 'int'>`

```python
print(type(list()))
```

返回 `<class 'list'>`

## Class Methods

```python
import random
class Hat:
    def __init__(self):
        self.houses = ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"] 
    def sort(self, name):
        print(name, "is in", random.choice(self.houses))
hat = Hat()
hat.sort("Harry")
```

a list 存储在 self.houses 中

### @classmethod

`@classmethod` is a function that we can use to add functionality to a class as a whole

### class variables

class 中的所有 method 都可以用

houses 现在不是 instance variable（call self.houses），是 class variable（call cls.houses）

```python
import random
class Hat:
    houses = ["Gryffindor", "Hufflepuff", "Ravenclaw", "Slytherin"] 
    @classmethod
    def sort(cls, name):
        print(name, "is in", random.choice(cls.houses))
Hat.sort("Harry")
```

`<name of the class>.<method name>(<argument>)`

观察：我们把 `class` 和 `cls` 都去掉，也可以运行。所以 class 作用是 organize

### 改变执行顺序

```python
class Student:
    def __init__(self, name, house):
        self.name = name
        self.house = house  
    def __str__(self):
        return f"{self.name} from {self.house}"
    @classmethod
    def get(cls):
        name = input("Name: ")
        house = input("House: ")
        return cls(name, house)			#instantiate a student object
def main():
    student = Student.get()
    print(student)
```

通过 call cls 返回新的 student object

call this method(get) without instantiating a student object first（没有执行 `__init__`  和 `__str__`）

> instance methods operate on specific objects(eg. student, hat)
>
> class methos & class variables operate on the entire class(all objects)

## Inheritance

### Inheritance 继承

> Inheritance is, perhaps, the most powerful feature of object-oriented programming

在不同 class 之间共享 methods, variables, attributes 

```python
class Wizard:
    def __init__(self, name):
        if not name:
            raise ValueError("Missing name")
        self.name = name

class Student(Wizard):
    def __init__(self, name, house):
        super().__init__(name)
        self.house = house

class Professor(Wizard):
    def __init__(self, name, subject):
        super().__init__(name)
        self.subject = subject 
```

Professor 和 Student 里都需要初始化 name

`class Student(Wizard):` 在定义 Student class 的时候，先 inherit Wizard class 的内容

`super()` 指向 superclass⬆️ of this class

### exceptions

exceptions come in a heirarchy, where there are children, parent, and grandparent classes

```
BaseException
 +-- KeyboardInterrupt
 +-- Exception
      +-- ArithmeticError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- KeyError
      +-- NameError
      +-- SyntaxError
      |    +-- IndentationError
      +-- ValueError
 ...
```

## Operator Overloading

### Operator Overloading

`+` can be overloaded such that it can 做加法/连接字符串，我们可以自定义 `+` 的意义

[python: operator overloading](https://docs.python.org/3/reference/datamodel.html#special-method-names)

### 代码

分别加

```python
class Vault:
    def __init__(self, galleons=0, sickles=0, knuts=0): #默认值为0
        self.galleons = galleons
        self.sickles = sickles
        self.knuts = knuts
    def __str__(self):
        return f"{self.galleons} Galleons, {self.sickles} Sickles, {self.knuts} Knuts"
potter = Vault(100, 50, 25)
print(potter)
weasley = Vault(25, 50, 100)
print(weasley)

galleons = potter.galleons + weasley.galleons
sickles = potter.sickles + weasley.sickles
knuts = potter.knuts + weasley.knuts
total = Vault(galleons, sickles, knuts)
print(total)
```

### 优化

```python
object.__add__(self, other)
```

self + other 

```python
class Vault:
    def __init__(self, galleons=0, sickles=0, knuts=0): #默认值为0
        self.galleons = galleons
        self.sickles = sickles
        self.knuts = knuts
    def __str__(self):
        return f"{self.galleons} Galleons, {self.sickles} Sickles, {self.knuts} Knuts"
    def __add__(self, other):
        galleons = self.galleons + other.galleons
        sickles = self.sickles + other.sickles
        knuts = self.knuts + other.knuts
        return Vault(galleons, sickles, knuts)

potter = Vault(100, 50, 25)
weasley = Vault(25, 50, 100)
total = potter + weasley
print(total)
```

potter 传入 self，weasley 传入 other

教给 python `+` 两个 Vault 的意义
