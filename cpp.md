# 💻 OOP Through C++ Lab Manual



## 🧮 Experiment – 1  
### **Write a C++ program to find the sum of individual digits of a positive integer.**

#### **Source Code:**
```cpp
#include <iostream>
using namespace std;
int main() {
    int n, sum = 0, digit;
    cout << "Enter a positive integer: ";
    cin >> n;
    while (n > 0) {
        digit = n % 10;
        sum += digit;
        n /= 10;
    }
    cout << "Sum of digits = " << sum << endl;
    return 0;
}
````

#### **Output:**

```
Enter a positive integer: 2545
Sum of digits = 16
```

    

## 🔢 Experiment – 2

### **Write a C++ program to generate all the prime numbers between 1 and n, where n is a value supplied by the user.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
int main() {
    int n;
    cout << "Enter n: ";
    cin >> n;
    for (int i = 2; i <= n; i++) {
        int flag = 0;
        for (int j = 2; j * j <= i; j++) {
            if (i % j == 0) {
                flag = 1;
                break;
            }
        }
        if (flag == 0) cout << i << " ";
    }
    return 0;
}
```

#### **Output:**

```
Enter n: 20
2 3 5 7 11 13 17 19
```

    

## 🧮 Experiment – 3

### **Write a C++ program to find both the largest and smallest number in a list of integers.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
int main() {
    int n;
    cout << "Enter number of elements: ";
    cin >> n;
    int arr[n];
    for (int i = 0; i < n; i++) cin >> arr[i];
    int largest = arr[0], smallest = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > largest) largest = arr[i];
        if (arr[i] < smallest) smallest = arr[i];
    }
    cout << "Largest: " << largest << "\nSmallest: " << smallest;
    return 0;
}
```

#### **Output:**

```
Enter number of elements: 5
25
2
36
85
4
Largest: 85
Smallest: 2
```

    

## 🔼 Experiment – 4

### **Write a C++ program to sort a list of numbers in ascending order.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
int main() {
    int n;
    cout << "Enter number of elements: ";
    cin >> n;
    int arr[n];
    for (int i = 0; i < n; i++) cin >> arr[i];
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (arr[i] > arr[j]) {
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
    cout << "Sorted list: ";
    for (int i = 0; i < n; i++) cout << arr[i] << " ";
    return 0;
}
```

#### **Output:**

```
Enter number of elements: 4
85
69
100
4
Sorted list: 4 69 85 100
```

    

## 🧠 Experiment – 5

### **Write a Program to illustrate New and Delete Keywords for dynamic memory allocation.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
int main() {
    int n;
    cout << "Enter number of elements: ";
    cin >> n;
    int* arr = new int[n];
    for (int i = 0; i < n; i++) cin >> arr[i];
    cout << "Array elements: ";
    for (int i = 0; i < n; i++) cout << arr[i] << " ";
    delete[] arr;
    return 0;
}
```

#### **Output:**

```
Enter number of elements: 5
25
4
65
8
5
Array elements: 25 4 65 8 5
```

    

## ⚙️ Experiment – 6

### **Write a Program to Demonstrate the i) Operator Overloading. ii) Function Overloading.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }

class A {
public:
    int x, y;
    A(int a = 0, int b = 0) { x = a; y = b; }
    A operator+(A p) { return A(x + p.x, y + p.y); }
};

int main() {
    cout << "Function Overloading: " << add(5, 3) << ", " << add(2.5, 3.5) << endl;
    A p1(1, 2), p2(3, 4);
    A p3 = p1 + p2;
    cout << "Operator Overloading: (" << p3.x << "," << p3.y << ")";
    return 0;
}
```

#### **Output:**

```
Function Overloading: 8, 6
Operator Overloading: (4,6)
```

    

## 🧱 Experiment – 7

### **Write a Program to Demonstrate Friend Function and Friend Class.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
class Box {
private:
    int length;
public:
    Box(int l) : length(l) {}
    friend void showLength(Box b);
    friend class BoxPrinter;
};
void showLength(Box b) {
    cout << "Friend Function: Length = " << b.length << endl;
}
class BoxPrinter {
public:
    void print(Box b) {
        cout << "Friend Class: Length = " << b.length << endl;
    }
};
int main() {
    Box b1(30);
    showLength(b1);
    BoxPrinter printer;
    printer.print(b1);
    return 0;
}
```

#### **Output:**

```
Friend Function: Length = 30
Friend Class: Length = 30
```

    

## 🧩 Experiment – 8

### **Write a program with constructor function and another for destruction function.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
class A {
public:
    A() { cout << "Constructor called\n"; }
    ~A() { cout << "Destructor called\n"; }
};
int main() {
    A obj;
    return 0;
}
```

#### **Output:**

```
Constructor called
Destructor called
```

    

## 🧾 Experiment – 9

### **Write a program to explain function overloading with different number of arguments.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
void print(int a) {
    cout << "One number: " << a << endl;
}
void print(int a, int b) {
    cout << "Two numbers: " << a << ", " << b << endl;
}
int main() {
    print(5);
    print(3, 7);
    return 0;
}
```

#### **Output:**

```
One number: 5
Two numbers: 3, 7
```

    

## 🧮 Experiment – 10

### **Write a program to explain function overloading with type, order, and sequence of arguments.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;
void show(int a, double b) {
    cout << a << ", " << b << endl;
}
void show(double a, int b) {
    cout << a << ", " << b << endl;
}
int main() {
    show(5, 3.5);
    show(2.5, 7);
    return 0;
}
```

#### **Output:**

```
5, 3.5
2.5, 7
```

    

## ⚡ Experiment – 11

### **Write C++ programs that illustrate how the following forms of inheritance are supported:**

* Single Inheritance
* Multiple Inheritance
* Multilevel Inheritance
* Hierarchical Inheritance

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;

// Single Inheritance
class A { public: void displayA() { cout << "Single Inheritance\n"; } };
class B : public A { public: void displayB() { cout << "Derived from A\n"; } };

// Multiple Inheritance
class X { public: void showX() { cout << "Class X\n"; } };
class Y { public: void showY() { cout << "Class Y\n"; } };
class Z : public X, public Y {};

// Multilevel Inheritance
class C { public: void baseC() { cout << "Base Class C\n"; } };
class D : public C {};
class E : public D { public: void derivedE() { cout << "Derived from D\n"; } };

// Hierarchical Inheritance
class Parent { public: void parentShow() { cout << "Parent Class\n"; } };
class Child1 : public Parent {};
class Child2 : public Parent {};

int main() {
    B objB; objB.displayA(); objB.displayB();
    Z objZ; objZ.showX(); objZ.showY();
    E objE; objE.baseC(); objE.derivedE();
    Child1 c1; c1.parentShow();
    Child2 c2; c2.parentShow();
    return 0;
}
```

#### **Output:**

```
Single Inheritance
Derived from A
Class X
Class Y
Base Class C
Derived from D
Parent Class
Parent Class
```

    

## ⚙️ Experiment – 12

### **a) Write a Program Containing a Possible Exception. Use a Try Block to Throw it and a Catch Block to Handle It Properly.**

### **b) Write a Program to Demonstrate the Catching of All Exceptions.**

#### **Source Code:**

```cpp
#include <iostream>
using namespace std;

int main() {
    try {
        int a, b;
        cout << "Enter two numbers: ";
        cin >> a >> b;
        if (b == 0)
            throw "Division by zero not allowed!";
        cout << "Result: " << a / b << endl;
    }
    catch (const char* msg) {
        cout << "Error: " << msg << endl;
    }

    // Catching all exceptions
    try {
        throw 10;
    }
    catch (...) {
        cout << "Caught an unknown exception!" << endl;
    }

    return 0;
}
```

#### **Output:**

```
Enter two numbers: 8 0
Error: Division by zero not allowed!
Caught an unknown exception!
```
