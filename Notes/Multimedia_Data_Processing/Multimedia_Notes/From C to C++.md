# From C to C++

# C++ is a oriented language model

C++ is an **object-oriented programming (OOP) language** that incorporates key principles of OOP:

- **Encapsulation** → Hides the internal details of an object and exposes only what is necessary through well-defined interfaces.
- **Inheritance** → Allows the creation of new classes (subclasses) based on existing classes (superclasses), reusing and extending their behavior.
- **Polymorphism** → Enables treating objects of different classes uniformly if they share a common interface.

<aside>
💡

**Rule of four** 

It suggests that if a class requires **any** of the following special member functions, it should implement **all four** to properly manage resource ownership:

1.	**Constructor** (`MyClass()`) → Initializes resources when an object is created.

2.	**Copy Constructor** (`MyClass(const MyClass& other)`) → Creates a deep copy of another object to prevent shared ownership of dynamic resources.

3.	**Copy Assignment Operator** (`MyClass& operator=(const MyClass& other)`) → Assigns an existing object by copying data, ensuring proper resource management.

4.	**Destructor** (`~MyClass()`) → Cleans up allocated resources to prevent memory leaks.

</aside>

---

# 1. Initialisation vs Assignment

- **Initialisation →** we have a pointer to the same piece of memory, resulting in a shallow copy.
    
    ![image.png](From%20C%20to%20C++/image.png)
    
- **Assignment →** copies both the structure and the actual data, creating a fully independent duplicate, resulting in a deep copy
    
    ![image.png](From%20C%20to%20C++/image%201.png)
    

![Shadow copy → Initialisation ](From%20C%20to%20C++/image%202.png)

Shadow copy → Initialisation 

![Deep copy → Assignment](From%20C%20to%20C++/image%203.png)

Deep copy → Assignment

---

# 2. Copy constructor

<aside>
🚨

**Problem**

If we copy a struct, we need to ensure that its data is also copied properly. Otherwise, when the original struct is no longer used, its destructor may free the memory, leading to issues if the copied struct still references the same data.

So if vector used shallow copying:

1.  When passing it by value, a copy of the object would be made, but only the pointer to the data would be copied, not the actual data.
2.  When the function ends, the destructor of the copied vector would free the shared memory.
3. Later, when the original vector is destroyed, it would try to free the same memory again, causing undefined behavior (double free error).
</aside>

To solve this problem, we use the **copy constructor**, ensuring that a deep copy is performed instead of a shallow copy.

![image.png](From%20C%20to%20C++/image%204.png)

When we use:

![image.png](From%20C%20to%20C++/95072396-c8c3-46d2-9168-8edf8f26125c.png)

We **implicitly call the copy constructor.**

---

# 3. References vs Pointers

In the **copy constructor**, we use a reference.

- In **C, references do not exist**, only pointers do.
- In **C++, both exist**

<aside>
💡

**Reference →** is an alias for another variable. It is not a pointer but simply another **name** for the same memory location.

**Pointer →** is a variable that **stores the address of another variable**.

</aside>

| Characteristic  | **Reference (&)** | **Pointer (*)** |
| --- | --- | --- |
| **How to declare?** | `int& ref = x;` | `int* ptr = &x;` |
| **What does it store?** | An alias of a variable | A memory address |
| **Must be initialized immediately?** | ✅ Yes | ❌ No |
| **Can it be NULL?** | ❌ No | ✅ Yes (`ptr = nullptr;`) |
| **Can it be reassigned?** | ❌ No | ✅ Yes (`ptr = &y;`) |
| **How to access the value?** | `ref (without *)` | `*ptr` |

For example, in the **print function,** we  use references to:

- Avoid unnecessary copying.
- Prevent accidental modifications (const guarantees read-only access).

![image.png](From%20C%20to%20C++/image%205.png)

---

# 4. Reference Inizialization

If you want to initialize a reference in a constructor, you cannot assign a new value to **`os_`** because it’s a reference:

![image.png](From%20C%20to%20C++/image%206.png)

Instead, you should use **`os_(os)`** to initialize it:

![image.png](From%20C%20to%20C++/image%207.png)

---

# 5. C**opy Assignment Operator (operator=)**

![image.png](From%20C%20to%20C++/image%208.png)

In C, using `=` to copy a struct creates a **shallow copy**, duplicating only primitive fields. This implementation ensures a **deep copy** by properly managing dynamic memory. Without it, the default assignment would copy only pointers, leading to **double free errors** or **memory leaks**. 

Returning `vector&` maintains standard assignment behavior, allowing chaining and avoiding unnecessary copies:

![image.png](From%20C%20to%20C++/image%209.png)

<aside>
🚨

**Why is Self-Assignment Dangerous (n`umbers = numbers`)?**

If `operator=` did **not** check for self-assignment, this could happen:

1.	`free(data_)` would delete existing memory.

2.	The copy loop would then try to access `data_`, which is now invalid.

3.	The program would crash.

</aside>

<aside>
📌

**Difference between C and C++**

If we write:

![image.png](From%20C%20to%20C++/image%2010.png)

In C++, the value of x remains unchanged, and **the expression returns x as a reference (not the address!),** instead in C the operation returns the **assigned value**.

</aside>

---

# 6. RVO/NRVO optimization

Returning a local object from a function **can lead to unnecessary copies**, which impact performance. However, **modern C++ compilers apply Return Value Optimization (RVO) and Named Return Value Optimization (NRVO)** to eliminate this overhead.

If **NRVO is supported**, the compiler **constructs v directly in the caller’s memory**, avoiding both **copying and moving**.

<aside>

**Example:**

![image.png](From%20C%20to%20C++/image%2011.png)

Without RVO/NRVO, returning v would require:

1. Creating v inside read
2. Copying or moving v into the caller (e.g., main)
3. Destroying v inside read
</aside>

---

# **7. Move Constructor: Handling Cases Without RVO**

If the compiler **does not** apply RVO, **a move constructor prevents deep copies**, transferring resources instead.

![image.png](From%20C%20to%20C++/image%2012.png)

Instead of copying the data, it **steals the pointer** from other, avoiding unnecessary allocations and deallocations. The original object (other) is left in a valid but empty state.

![image.png](From%20C%20to%20C++/image%2013.png)

<aside>
💡

**R-value References (&&)**

`&&` is an **R-value reference**, meaning it refers to **temporary objects** that are about to be destroyed.

</aside>

Example of move constructor usage:

![image.png](From%20C%20to%20C++/image%2014.png)

---

# 8. Swap

We have a **standard library function** called swap, available through `#include <utility>`, which allows us to efficiently exchange resources.

Instead of **freeing data_ and reallocating memory**, we can use swap to efficiently **transfer ownership** of resources.

![image.png](From%20C%20to%20C++/image%2015.png)

Alternatively, we could manually swap pointers like this:

![image.png](From%20C%20to%20C++/d80be52f-92b9-4c67-9ca6-a52f8e5ad343.png)

However, using `std::swap` makes the code **cleaner and more maintainable**.

---

# 9. Template

A **template** in C++ enables **generic programming**, meaning that the same class or function can work with different types without rewriting the logic.

![image.png](From%20C%20to%20C++/image%2016.png)

---

# **10. Operator[]**

**To allow modification of the vector** we need 2 versions of the `operator[]` function:

- **Const version** (for read-only access):
    
    ![image.png](From%20C%20to%20C++/image%2017.png)
    
- **Non-const version** (for modification):
    
    ![image.png](From%20C%20to%20C++/ed39efc0-4b61-4101-add4-44cb65fcde73.png)
    

<aside>
🚨

**Placement of const**

- **const (before function) →** the function return a constant value/reference **→** Prevents modification of the returned value.
- **const (after function) →** This means the function does not modify any member variables **→** Guarantees that the function does not change the object’s state.
</aside>

---

# 11. New vs Malloc

- **`new`** → Allocates memory **and** calls the constructor.
- **`malloc`** → Allocates memory **but does not call the constructor**.

<aside>
💡

**Allocating and Deallocating Arrays of Objects**

- Single object:
    
    ![image.png](From%20C%20to%20C++/image%2018.png)
    
- Array of objects:
    
    ![image.png](From%20C%20to%20C++/image%2019.png)
    

**N.B.** C++ use two different ways to allocate memory because if I allocate multiple objects separately (instead of using an array), I might need to store a pointer for each allocation, which could lead to extra memory overhead.

</aside>

---

# 12. Iterators

An **iterator** in C++ is an object that **points to elements** inside a **container** and allows you to traverse through them **sequentially**, similar to how pointers work with arrays.

We can implement them in different ways:

1. **Using Raw Pointers (Old-School C Style)**
    
    ![image.png](From%20C%20to%20C++/image%2020.png)
    
    <aside>
    🚨
    
    **Issues:**
    
    - Works only if v is **not empty** (v[0] is undefined otherwise).
    - If v is reallocated (e.g., v.push_back()), start and stop become invalid.
    </aside>
    
2. **Using Explicit Iterators (C++98 Style)**
    
    ![image.png](From%20C%20to%20C++/image%2021.png)
    
    <aside>
    🚨
    
    **Verbose**: typing `std::vector<int>::const_iterator` everywhere is annoying.
    
    </aside>
    
3. **Using auto (C++11 Style)**
    
    ![image.png](From%20C%20to%20C++/image%2022.png)
    
    We can do it in one single loop:
    
    ![image.png](From%20C%20to%20C++/image%2023.png)
    
4. **The Best Way: Range-Based for Loop (C++11+)**
    
    ![image.png](From%20C%20to%20C++/dadaef14-7ab1-48d0-a453-dd20bf7fd8eb.png)
    
    If you need to **modify elements**, drop const:
    
    ![image.png](From%20C%20to%20C++/image%2024.png)
    

---

# 13. Sorting in C++

C++ provides two common ways to sort arrays/containers:

- **qsort (C-style)**
    
    ![image.png](From%20C%20to%20C++/image%2025.png)
    
- **std::sort (C++-style)** → It uses **iterators**, meaning it works with any container (not just arrays). The compiler can optimize it better than qsort.
    
    ![image.png](From%20C%20to%20C++/image%2026.png)
    
    <aside>
    🚨
    
    **Why Use** `begin(numbers)`**,** `end(numbers)` **instead of** `numbers.begin()`**,** `numbers.end()`**?**
    
    - `numbers.begin()` and `numbers.end()` are **member functions** of `std::vector<int>`.
    - `begin(numbers)` and `end(numbers)` are **free functions** in `<iterator>`, which work for both **arrays and containers**.
    </aside>
    
    If you don’t provide a comparison function, std::sort **defaults** to:
    
    ![image.png](From%20C%20to%20C++/image%2027.png)
    

## **13.1 Why doesn’t std::sort work with std::vector<widget>?**

Sorting is obviously an algorithm, but the main issue is that `std::sort` requires a **sorting criterion**. However, the widget class does not have a comparison operator defined.

- **Solution 1: Define operator< in widget**
    
    ![image.png](From%20C%20to%20C++/image%2028.png)
    
- **Solution 2: Using a Custom Comparison Function (compare)**
    
    ![image.png](From%20C%20to%20C++/image%2029.png)
    
- **Solution 3: Using a Functor (Comparator) →** A **functor** is an object that behaves like a function.
    
    ![image.png](From%20C%20to%20C++/image%2030.png)
    
    <aside>
    💡
    
    **Why use a functor?**
    
    - It can maintain an **internal state**
    - t allows **custom sorting behavior** via constructors or member variables.
    </aside>
    
- **Solution 4: Using a Lambda Function →** Lambda functions offer a more **concise and readable** way to specify a custom sorting criterion.
    
    ![image.png](From%20C%20to%20C++/image%2031.png)
    

<aside>
💡

 **Simplifying std::sort with a Macro**

Writing begin(container), end(container) repeatedly is annoying. A macro can help:

![image.png](From%20C%20to%20C++/image%2032.png)

</aside>

---

# 14. Input/Output in C++

C++ provides various ways to handle input and output, mainly through **streams**.

1. **Standard Output (`std::cout`, `std::cerr`) →** The `<<` operator is used to append data to an output stream. It is evaluated **from left to right** and returns the same stream, allowing multiple elements to be chained.
    
    ![image.png](From%20C%20to%20C++/image%2033.png)
    
2. **Standard Input (`std::cin`) →** `std::cin` reads input from the keyboard.
    
    ![image.png](From%20C%20to%20C++/image%2034.png)
    
    **N.B.** `std::cin` ignores whitespaces, so for reading full strings, prefer `std::getline`.
    
3. **File I/O (`<fstream>`) →** you can read and write files using `std::ifstream` (input file stream) and `std::ofstream` (output file stream).
    - **Reading from File**
        
        ![image.png](From%20C%20to%20C++/image%2035.png)
        
    - **Writing to a File**
        
        ![image.png](From%20C%20to%20C++/image%2036.png)
        
        <aside>
        📌
        
        **Why std::ios::binary?**
        
        On Windows, it prevents automatic CRLF (\r\n) conversion.
        
        </aside>
        

<aside>
🚨

**Error Handling in Streams**

You can check if a file was successfully opened using:

- `input.fail()` → error opening the file
- `input.good()` → the stream is in a good state
- `!input` → equivalent to fail()

Example:

![image.png](From%20C%20to%20C++/image%2037.png)

</aside>

## **14.1 Reading and Writing with std::istream_iterator and std::ostream_iterator**

C++ provides stream iterators to efficiently read and write data.

- **Reading with std::istream_iterator**
    
    ![image.png](From%20C%20to%20C++/image%2038.png)
    
    The compiler may interpret `std::vector<int> numbers(...)`; as a function declaration (→ **“Most vexing parse”** problem). To avoid this, use `{}` instead of `()`:
    
    ![image.png](From%20C%20to%20C++/image%2039.png)
    
- **Writing with std::oftream_iterator**
    
    ![image.png](From%20C%20to%20C++/image%2040.png)
    

---

# 15. String formatting

C++ provides several ways to format strings, allowing us to print multiple values in a string without knowing its size in advance.

1. **The Problem with Traditional Concatenation →** before C++20, string formatting relied on `std::cout` or s`td::stringstream`, which could be annoying:
    
    ![image.png](From%20C%20to%20C++/image%2041.png)
    
2. **Using std::format (C++20) →** `std::format` uses `{}` as placeholders (similar to Python’s f-strings), avoids type conversion issues (e.g., manually converting numbers to strings), and supports advanced formatting like alignment, padding, and numeric precision.
    
    ![image.png](From%20C%20to%20C++/image%2042.png)
    
3. **Using std::print (C++23) →** C++23 introduces `std::print`, which works like `std::format` but prints directly to stdout without requiring `std::cout`:
    
    ![image.png](From%20C%20to%20C++/image%2043.png)
    

---

# **16. Order Containers: std::set and std::multiset**

- **`std::set<T>`** → sorted set **without duplicates**.
- **`std::multiset<T>`** → sorted set **with duplicates**.

![image.png](From%20C%20to%20C++/image%2044.png)

**Searching for a Value:**

![image.png](From%20C%20to%20C++/image%2045.png)

---

# 17. Casting

C++ provides four main types of type casting:

1.  **static_cast →** Used for safe conversions between compatible types.
    
    ![image.png](From%20C%20to%20C++/image%2046.png)
    
    <aside>
    🚨
    
    Cannot be used between unrelated pointer types (e.g., int* to double*):
    
    ![image.png](From%20C%20to%20C++/image%2047.png)
    
    </aside>
    
2. **reinterpret_cast** **→** Useful for treating a pointer as another pointer type (does not convert data, only changes how memory is interpreted).
    
    ![image.png](From%20C%20to%20C++/image%2048.png)
    
3. **const_cast →** Used for adding or removing const from a variable.
    
    ![image.png](From%20C%20to%20C++/image%2049.png)
    
4. **dynamic_cast →** Used for converting between classes in an inheritance hierarchy (only with polymorphism).
    
    ![image.png](From%20C%20to%20C++/image%2050.png)
    

| **Cast** | **Main Use Case** | **Safe?** | **Example** |
| --- | --- | --- | --- |
| **`static_cast<T>()`** | Converting between compatible types | ✅ | double -> int |
| **`reinterpret_cast<T*>()`** | Forced pointer type conversion | ⚠️ | int* -> char* |
| **`const_cast<T>()`** | Removing const from a variable | ⚠️ | const int* -> int* |
| **`dynamic_cast<T*>()`** | Safe casting between polymorphic classes | ✅ | Base* -> Derived* |

---

# 18. Struct vs Class

<aside>
💡

- When you **inherit from a struct**, the default visibility is **public**.
- When you **inherit from a class**, the default visibility is **private**.
</aside>

You **can** explicitly use **`public:`** in a class to make it behave like a struct:

![image.png](From%20C%20to%20C++/image%2051.png)

![image.png](From%20C%20to%20C++/image%2052.png)

---

# 19. — >

- **Problem:**
    
    ![image.png](From%20C%20to%20C++/image%2053.png)
    
    **Problem**: This causes an **endless loop** because `i` is of type `size_t`, which is an **unsigned integer**. When i becomes 0, the condition `i >= 0` will always be true (since **size_t is unsigned**, and 0 is always greater than or equal to 0). As a result, the loop will keep decrementing `i` into negative values (which wrap around and become very large values) and never exit.
    
- **First possible solution:**
    
    ![image.png](From%20C%20to%20C++/image%2054.png)
    
- **Second possibile solution:**
    
    ![image.png](From%20C%20to%20C++/image%2055.png)
    
- **Third possible solution:**
    
    ![image.png](From%20C%20to%20C++/image%2056.png)
    

---

# 20. Inheritance

Inheritance in C++ is a mechanism that allows one class (the derived class) to inherit properties and behaviours (fields and methods) from another class (the base class).

Example:

![image.png](From%20C%20to%20C++/image%2057.png)

This is a **base class** called shape, It has four **attributes:**

- `x_`, `y_`: Coordinates of the shape.
- `c_`: The character used to represent the shape.
- `name_`: The name of the shape.
- The constructor initializes these attributes.
- The function `draw()` is marked as **pure virtual**, making shape an **abstract class** (we cannot instantiate it directly).

<aside>
💡

**Virtual →** keyword that tells the compiler: **“This method can be overridden by derived classes.”** Virtual functions allow **polymorphic behavior**, where the actual method called is determined at runtime based on the **real type** of the object, not just its reference type.

</aside>

The class **point** inherits from shape:

![image.png](From%20C%20to%20C++/image%2058.png)

- `: public shape` means that point **inherits** all the attributes (`x_, y_, c_, name_`) and methods of shape.
- It **extends** the functionality by adding a unique pointer what_.
- It **implements** the `draw()` method that was declared as pure virtual in the base class.

<aside>
💡

**Override →** Ensures that you are indeed overriding a **virtual function** from the base class. It helps catch mistakes where:

- The method name is misspelled.
- The parameter types are incorrect.
- You accidentally use a different signature.
</aside>

### **Polymorphism in Action**

In the main function:

![image.png](From%20C%20to%20C++/image%2059.png)

Each shape (point, line, rectangle, circle) is drawn using its own `draw()` method. This is possible because of **polymorphism**:

- Each object is referenced as a shape pointer.
- The correct draw() method is called at runtime based on the **actual type** of the object.

---

# Template

## BitWriter

```cpp
struct bit_writer {
    std::ostream& os_;
    uint8_t buffer_ = 0;
    size_t n_ = 0;

    bit_writer(std::ostream& os) : os_(os) {}
    
    void writebit(uint32_t bit) {
        buffer_ = (buffer_ << 1) | (bit & 1);
        n_++;
        if (n_ == 8) {
            os_.write(reinterpret_cast<const char*>(&buffer_), sizeof(buffer_));
            n_ = 0;
        }
    }

    void write_n_bit(uint32_t u, size_t n) {
        while (n-- > 0) {
            writebit(u >> n); //BIG-ENDIAN
            //writebit(u & 1); //LITTLE-ENDIAN
            //u>>=1;
        }
    }

    ~bit_writer() {
        while(n_ > 0) writebit(0);
    }
};

void write_int11(std::ifstream& ifs,std::ofstream& ofs,bit_writer& b) {
    uint32_t c=0;
    while (ifs >> c) {
        b.write_n_bit(c,11);
    }
}
```

## BitReader

```cpp
struct bit_reader {
    uint8_t buffer = 0;
    size_t n_ = 0;
    std::istream& ifs_;

    bit_reader(std::istream& ifs) : ifs_(ifs) {}

    uint32_t read_bit() {
        if (n_ == 0) {
            ifs_.read(reinterpret_cast<char*>(&buffer), sizeof(buffer));
            n_=8;
        }
        n_--;
        return (buffer >> n_) & 1;
    }

     std::istream& read_n_bits(uint32_t& u,size_t n) {
		    u=0;
        //size_t shift = 0;
        while (n-- > 0) {
            u = (u << 1) | read_bit(); //read from MSB to LSB
            //u |= (read_bit() << shift); // scrivi il bit nel posto giusto
            //shift++;
        }
        return ifs_;
    }
};
```

## Write in hexadecimal

```cpp
std::cout << std::hex << std::setw(2) << std::setfill('0') << (static_cast<unsigned int>(str_elem[j + i]) & 0xFF);
```

## Images

```cpp
using rgb = std::array<uint8_t, 3>;
template<typename T>
struct mat {
	size_t rows_, cols_;
	std::vector<T> data_;

	mat(size_t rows = 0, size_t cols = 0) 
		: rows_(rows), cols_(cols), data_(rows*cols) {}

	auto rows() const { return rows_; }
	auto cols() const { return cols_; }
	auto size() const { return data_.size(); }

	template<typename Self> // Deducing this
	auto&& operator()(this Self&& self, size_t r, size_t c) {
		return self.data_[r * self.cols_ + c];
	}

	auto rawdata() {
		return reinterpret_cast<char*>(data_.data());
	}
	auto rawdata() const {
		return reinterpret_cast<const char*>(data_.data());
	}
	auto rawsize() const { return size() * sizeof(T); }
}
```

```cpp
ifs_.ignore(std::numeric_limits<std::streamsize>::max(),'\n');
```

# ByteSwap

```cpp
#include <bit>
#include <cstdint>

uint16_t swapped = std::byteswap(uint16_t{0x1234});
uint32_t swapped32 = std::byteswap(uint32_t{0x12345678});
```

In C++ le variabili globali vengono preinizializzatte prima del main. Il codice dei scostruttori vengono inizializati prima del main, ma non possiamo forzare l’ordine di inizializzazione dei costruttori.

SI parla di static inizialization order fiasco. 

```cpp
for (auto it = table.find(header.id_); it != table.end()) {
 //----
}
//for con inizializzazione dentro, possibile grazie alle ultime versioni di C++
```

#parameter = “parameter”

name##RegisterStruct → per concantenare