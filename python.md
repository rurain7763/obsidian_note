#### Variable
타입 지정이 필요없다
```python
name = "John"
age = 20
is_new = True
```
#### Input
```python
name = input("What is your name? ")
```
#### Output
```python
print("Hello " + name)
```
#### Type casting
```python
age = input("What is your age? ")
new_age = age + 1 # Type error : str + int
new_age = int(age) + 1 # Type casting
```
- int()
- float()
- str()
- bool()
- etc
```python
first_num = float(input("First: "))
second_num = float(input("Second: "))
print("Sum: " + str(first_num + second_num))
```