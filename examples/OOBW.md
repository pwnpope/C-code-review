
## Out Of Bounds Write

- Often used to describe the consequences of `writing to memory outside the bounds of a buffer`, or to memory that is invalid, when the root cause is something other than a sequential copy of excessive data from a fixed starting location.
	- This may include issues such as `incorrect pointer arithmetic, accessing invalid pointers due to incomplete initialization or memory release`, etc.

- The product writes data past the end, or before the beginning, of the intended buffer.

- This can result in `corruption of data, a crash, or code execution`. 
	- The product may `modify an index or perform pointer arithmetic` that references a memory location that is outside of the boundaries of the buffer.
	- A subsequent write operation then produces undefined or unexpected results.

---

#### Examples of out-of-bounds write & cases where it would occur

- 1.  Writing beyond the end of an array: If a program writes data beyond the end of an array, it can cause an out-of-bounds write. For example:
```c
int buffer;
scanf("%d", &buffer); // user input

int array[10];
array[11] = buffer; // out-of-bounds write
```

- 2. Writing to a NULL pointer: If a program writes data to a NULL pointer, it can cause an out-of-bounds write. For example:
```c
int buffer;
scanf("%d", &buffer); // user input

int *p = NULL;
*p = buffer; // out-of-bounds write
```

- 3. Writing to a dangling pointer (use-after-free): If a program writes data to a pointer that has been freed or otherwise invalidated, it can cause an out-of-bounds write. For example:
```c
int buffer;
scanf("%d" &buffer); // user input

int *p = malloc(1337); 
int *q = p; // copy the address of the ptr p which held the malloc to the pointer q

free(p);  // the ADDRESS of the malloc which was pointed to p is now freed, it should be noted that q and p point to the same address

*q = buffer; // dangling pointer being written too
```