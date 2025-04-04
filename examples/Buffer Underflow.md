- The product writes to a buffer using an index or pointer that references a memory location prior to the beginning of the buffer.

- This typically occurs when a `pointer or its index is decremented to a position before the buffer`, when pointer arithmetic results in a position before the beginning of the valid memory location, or when a negative index is used.

- Some prominent vendors and researchers use the term "buffer under-run". 
	- "Buffer underflow" is more commonly used, although both terms are also sometimes used to describe a buffer under-read.

- all of the ways a buffer underflow can occur:
	- `1`) Reading from a file or network stream: If a program reads data from a file or network stream without checking the length of the data first, it could cause a buffer underflow.

	- `2`) Manipulating a string with a negative index: If a program manipulates a string with a negative index, it could cause a buffer underflow.

	- `3`) Using a pointer that points before the beginning of a buffer: If a program uses a pointer that points before the beginning of a buffer, it could cause a buffer underflow.

	- `4`) Calling a function with too few arguments: If a program calls a function with too few arguments, it could cause a buffer underflow if the function attempts to access arguments that were not passed.

	- `5`) Using an uninitialized buffer: If a program uses an uninitialized buffer, it could cause a buffer underflow if the buffer contains garbage data that is smaller than the intended data.

	- `6`) Allocating a buffer that is too small: If a program allocates a buffer that is too small to hold the data it needs to store, it could cause a buffer underflow if it attempts to access memory beyond the end of the buffer.

	- `6`)  Using a negative array index: If a program uses a negative index to access an array, it could cause a buffer underflow.


---
## 4 examples
```c
#include <stdio.h>

int main() {
  FILE *fp;
  char buffer[10];

  fp = fopen("file.txt", "r");
  if (fp == NULL) {
    printf("File not found\n");
    return 1;
  }

  // read 15 characters from the file into the buffer
  fread(buffer, 1, 15, fp);

  // print the contents of the buffer
  printf("Buffer contents: %s\n", buffer);

  fclose(fp);
  return 0;
}
```
- In this example, the program attempts to read 15 characters from a file into a buffer of size 10. Since the buffer is too small to hold all 15 characters, this will result in a buffer underflow. Depending on the content of the file, this could cause the program to read data from memory locations beyond the beginning of the buffer.

---
- Manipulating a string with a negative index: If a program manipulates a string with a negative index, it could cause a buffer underflow. For example, in C, a string can be accessed using array notation, but using a negative index will cause the program to access memory before the beginning of the string.
```c
#include <stdio.h>

int main() {
  char str[] = "hello";
  int index = -2;

  printf("Character at index %d: %p\n", index, str[index]);

  return 0;
}
```
- In this example, the program attempts to print the pointer at index-2 of the string "hello". Since the index is negative, this will cause the program to access memory before the beginning of the string, resulting in a buffer underflow.

---
- Using a pointer that points before the beginning of a buffer: If a program uses a pointer that points before the beginning of a buffer, it could cause a buffer underflow. For example, in C, a pointer can be used to access the elements of an array, but using a pointer that points before the beginning of the array will cause the program to access memory before the beginning of the buffer.
```c
#include <stdio.h>

int main() {
  int arr[5] = {1, 2, 3, 4, 5};
  int *ptr = arr - 1;

  // print the value at the address pointed to by ptr
  printf("Value at address %p: %d\n", ptr, *ptr);

  return 0;
}
```
- In this example, the program initializes an array of integers and a pointer that points before the beginning of the array. The program then attempts to print the value at the address pointed to by the pointer. Since the pointer points before the beginning of the array, this will cause the program to access memory before the beginning of the buffer, resulting in a buffer underflow.
