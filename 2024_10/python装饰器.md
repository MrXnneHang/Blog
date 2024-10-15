# Python装饰器

## 类里的常见装饰器：

### 1. @staticmethod
```python
class MyClass:
    @staticmethod
    def static_method(x, y):
        return f"Static method called. x + y = {x + y}"

# 使用类来调用静态方法
print(MyClass.static_method(5, 10))  
# 输出: Static method called. x + y = 15

# 使用实例来调用静态方法
instance = MyClass()
print(instance.static_method(7, 3))  
# 输出: Static method called. x + y = 10
```


### 2. @classmethod
```python
class MyClass:
    class_variable = "I am a class variable"
    
    def __init__(self, value):
        self.value = value
    
    @classmethod
    def class_method(cls):
        return f"Class method called. Class variable: {cls.class_variable}"

# 使用类来调用类方法
print(MyClass.class_method())  
# 输出: Class method called. Class variable: I am a class variable

# 使用实例来调用类方法
instance = MyClass(42)
print(instance.class_method())  
# 输出: Class method called. Class variable: I am a class variable
```

### 3. @property
```python
class SalesItem:
    def __init__(self, book_no='', units_sold=0, revenue=0.0):
        self.book_no = book_no
        self.units_sold = units_sold
        self.revenue = revenue
    
    @property
    def isbn(self):
        return self.book_no
```


前两种情况，可以直接类名调用，而不需要实例化。而@property则是实例化后调用。<br>

Access:<br>
@property: 可以访问self的属性<br>
@classmethod:需要写一个cls，不能访问self的属性。但可以访问类init前定义的属性。<br>
@staticmethod:不能访问self的属性，也不能访问类init前定义的属性。<br>

## 自定义装饰器:

### 1. 无参数装饰器
```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()

# 输出:
# Something is happening before the function is called.
# Hello!
# Something is happening after the function is called.
```

### 2. 有参数装饰器
```python
def my_decorator_with_args(number):
    def my_decorator(func):
        def wrapper(*args, **kwargs):
            print(f"Something is happening before the function is called. {number}")
            func(*args, **kwargs)
            print(f"Something is happening after the function is called. {number}")
        return wrapper
    return my_decorator

@my_decorator_with_args(42)
def say_hello():
    print("Hello!")

say_hello()

# 输出:
# Something is happening before the function is called. 42
# Hello!
# Something is happening after the function is called. 42
```

它可以用于，循环，计时，日志等等。<br>

本身就是嵌套函数。可以少掉很多重复代码，而且美观。在pyqt里也会见到@app,Flask里也会见到@app.route。<br>

好像没什么作用，也没什么深入，还只是语法层面。只是以前不了解，这里记录一下。<br>
