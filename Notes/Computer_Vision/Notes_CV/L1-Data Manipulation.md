# L1-Data Manipulation

Done?: Done
Select: lab

# 1. What is a Tensor?

A **tensor** is a multi-dimensional array used in PyTorch (similar to NumPy arrays). They generalize scalars, vectors, and matrices to higher dimensions. Notationally, they are represented using capital letters (e.g., **X, Y, Z**) with an indexing system similar to matrices. In general, a tensor consists of:

- **data**
- **metadata**, which include information like:
    - **shape** (number of dimensions and their sizes)
    - **data type** (e.g., float32, int64)
    - **device** where it’s stored (CPU or GPU memory)
    - **Strides**, which define how elements are stored in memory, and an **offset**, which affects indexing
    
    **N.B.** Data type in a tensor is **constant** after its creation.
    

---

# 2. Basic Tensor Commands in PyTorch

| **Command** | **Operation** | **Example** |
| --- | --- | --- |
| **`import torch`** | Import PyTorch library |  |
| **`torch.arange(start,end)`** | Create a 1D tensor with values in a range | x = torch.range(4) → #Output tensor([0, 1, 2, 3]) |
| **`tensor.shape`** | Get tensor shape | x.size() →#Output torch.Size([4]) |
| **`tensor.numel()`** | Get total number of elements in tensor | x.numel() → #Output 4 |
| **`tensor.reshape(dim1, dim2, ...)`** | To change the shape of a vector without altering the number of elements and their values.  |  |
| **`tensor.reshape(-1, dim2, ...)`** | Automatically **figure out that size for you**, so the total number of elements stays the same. |  |
| **`torch.zeros(shape)`** | Create a tensor filled with zeros | torch.zeros((2,3,4)) |
| **`torch.ones(shape)`** | Create a tensor filled with ones | torch.ones((2,3,4)) |
| **`torch.randn(shape)`** | Create a tensor with random values from a normal distribution | torch.randn(3,4) |
| **`torch.tensor(data)`** | Create a tensor from a list or array | torch.tensor([[2, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]]) |

---

# 3. Strided Representation

Tensor are stored **contiguously** in memory, meaning they are written in sequence, typically row by row.

<aside>

**Example:**

If the tensor stores **32-bit integers (4 bytes each)**, each element will be located at a memory address **4 bytes apart** from the previous one. However, since memory is just a continuous block of data, we need **extra metadata** to remember the tensor’s **shape (dimensions)**.

![image.png](L1-Data%20Manipulation/image.png)

</aside>

To locate an element in **physical memory** based on its **logical position in the tensor**, we use **strides:**

- A **stride** tells us how many memory locations we need to **skip** to reach the next element in a given dimension
- To compute the memory location of any element, we **multiply each index by its corresponding stride** and **sum the result**

![image.png](L1-Data%20Manipulation/image%201.png)

This allows PyTorch (or any framework) to efficiently access elements without having to reshape or reorder data in memory.

---

# 4. Tensor Operations in PyTorch

- **Elementwise Operation** → is applied to tensors of the same shape
    - **Unary** operations like **`torch.abs(x)`** apply to each element independently. Unary elementwise operations **preserve shape**.
    - **Binary** operations (`+`, `-`, `*`, `/`) act on corresponding elements of two tensors. Binary elementwise operations **result in the same shape as inputs**.
        
        ![image.png](L1-Data%20Manipulation/image%202.png)
        
    
    **N.B.** **Hadamard product** (elementwise multiplication) is denoted by $\odot$.
    
- **Reduction Operations**
    - **`sum()`**, **`mean()`**, and similar functions **reduce all axes by default**, outputting a scalar. We can also specify an axis to reduce along a specific dimension:
        
        ![image.png](L1-Data%20Manipulation/image%203.png)
        
- **Non-Reduction Operations** → ****keeping dimensions after reduction with **`keepdims=True`** preserves the number of axes.
    
    ![image.png](L1-Data%20Manipulation/image%204.png)
    
- **Dot Products** → ****computes **weighted sum or similarity between vectors**  

$x^\top y = \sum_{i=1}^{d} x_i y_i$. It can be used to measure the **similarity between vectors** because the **angle** between them is more important than their length. By normalizing the vectors, we obtain the **cosine similarity**: 

$\cos(\theta) = \frac{x^\top y}{\|x\| \|y\|}$.
    
    ![image.png](L1-Data%20Manipulation/image%205.png)
    
- **Tensor Concatenation** → ****stacks tensors along a specified axis using **`torch.cat()`:**
    
    ![image.png](L1-Data%20Manipulation/image%206.png)
    

---

# 5. Broadcasting

<aside>
💡

**Broadcasting** allows arrays (or tensors) with different shapes to be used in elementwise operations. This mechanism works in the following way:

1. Expand one or both arrays by copying elements appropriately so that after this transformation, the two tensors have the same shape.
2. Carry out the elementwise operations on the resulting arrays.
</aside>

## 5.1. Broadcasting rules

1. If the two arrays don’t have the same ranks (=number of axis), add 1s to the beginning of the smaller shape until they have the same length.
2. The 2 arrays are said to be compatibile if they have the same size in a dimension, or if one of the arrays has size 1 in that dimension.
3. Two arrays can be broadcast together only if they are compatible in **all dimensions**.
4. After broadcasting, the final shape is the **elementwise maximum** of the two shapes.
5. If one array has size 1 in a dimension and the other has a larger size, the smaller array is **copied (repeated)** along that dimension to match the larger one.

Example:

![image.png](L1-Data%20Manipulation/image%207.png)

**N.B.** During the “expansion” we **never allocate new memory**

---

# 6. Indexing and slicing

Elements in a tensor can be accessed by **index**. The first element has index 0, and ranges include the first but exclude the last element. Negative indices access elements from the end. 

**Slicing** extracts multiple elements. For example, **`X[0:2, :]`** selects the first two rows, while **`X[:, 0]`** extracts the first column:

![image.png](L1-Data%20Manipulation/image%208.png)

**N.B.** When you **slice** a tensor, you get a **view** of the original tensor, not a copy. So, modifying the elements of the slice also modifies the original tensor.

---

# 7. View

<aside>
💡

A **view** in PyTorch is a new tensor that shares the same data as the original tensor but with a different shape or subset of elements. Since views do not allocate new memory, modifying a view **also modifies the original tensor**. 

When you create a view of a tensor, PyTorch does **not** create a new copy of the data. Instead, it just creates a new way of accessing the **same memory**.

Examples:

![image.png](L1-Data%20Manipulation/image%209.png)

![image.png](L1-Data%20Manipulation/image%2010.png)

**N.B.** Normally, a tensor’s memory is **contiguous.** However, slicing or advanced views may break this order. Since some PyTorch operations want contiguous inputs, we can use **`.contiguous()`** to recover contiguity.

</aside>

## 7.1. Commands To Create View

- **`torch.as_strided(input, size, stride, storage_offset=0)`** → create a view of an existing torch by setting manually setting stride and offset.
- **`.view()`** → can be used to reshape a tensor. It raises an exception if:
    - the view I want to obtain has less elements than original
    - tensor is not contiguous
- **`.reshape()`** → can be used to reshape a tensor. If a tensor is contiguous in memory, it returns a view otherwise return a copy (under the hood it calls `.contiguous()`). It raises an exception if the view I want to obtain has less elements than original.

---

# 8. Numpy Bridge

PyTorch allows conversion between **Torch Tensors** and **NumPy arrays**, and they share **the same memory space**. This means that modifying one will **automatically** modify the other.

- **PyTorch → Numpy:**
    
    ![image.png](L1-Data%20Manipulation/image%2011.png)
    
- **Numpy → PyTorch:**
    
    ![image.png](L1-Data%20Manipulation/image%2012.png)
    

**N.B.** It works only if the tensor is on the CPU, otherwise is needed to move in CPU first. This because Numpy doesn’t support GPU operations and can’t read form GPU.

---