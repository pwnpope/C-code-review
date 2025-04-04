##### Out Of Bounds Read / buffer over-read

- The product reads data past the end, or before the beginning, of the intended buffer.

- Typically, this can allow attackers to read sensitive information from other memory locations or cause a crash. 
- A crash can occur when the code reads a variable amount of data and assumes that a sentinel exists to stop the read operation, such as a NULL in a string.
- The expected sentinel `might not be located in the out-of-bounds memory`, causing excessive data to be read, leading to a segmentation fault or a buffer overflow. 
- The product may `modify an index or perform pointer arithmetic` that references a memory location that is outside of the boundaries of the buffer. 
- A subsequent read operation then produces `undefined or unexpected results`.

- By reading out-of-bounds memory, an attacker might be able to get secret values, such as memory addresses, which can be bypass protection mechanisms such as ASLR in order to improve the reliability and likelihood of exploiting a separate weakness to achieve code execution instead of just denial of service.

---

#### Examples of out-of-bounds read & cases where it would occur
- 1.  Accessing an array element outside the bounds of the array:
```c
int buffer;
scanf("%d", &buffer); // user input

int arr[10];
int oobr = arr[buffer]; // out-of-bounds read as long as the user input is greater than 10

printf("%p", oobr);
```

- 2.  Accessing a struct member outside the bounds of the struct:
```c
struct my_struct {
    int x;
    int y;
};

int buffer;
scanf("%d", &buffer); // get index from user

struct my_struct s;
int member = *((int*)(&s + buffer)); // out-of-bounds read

printf("%p", member);
```
- In this example, a variable `s` of the `my_struct` type is declared and allocated on the stack. 
	- Then, an integer variable `member` is assigned the value of a memory location beyond the bounds of the `s` struct instance by performing pointer arithmetic.

- Specifically, the `&s + 1` expression calculates the address of the memory location immediately after the end of the `s` struct instance. 
	- This address is then cast to an `int` pointer and dereferenced to obtain the value of the integer at that location. Since the location is beyond the bounds of the `s` struct instance, this is an `out-of-bounds read.`

- Accessing a pointer outside the bounds of the memory it points to:
```c
int buffer;
scanf("%d", &buffer); // user input

int *p = malloc(10);
int oobr = p[buffer]; // out-of-bounds read

printf("%p", oobr);
```

- 4.  Accessing a function argument outside the bounds of the function call:
```c
void foo(void **s) {
	long buffer;
	scanf("%d", &buffer); // anything past 12 would trigger oobr
    char c = s[buffer]; // out-of-bounds read
    printf("%p", c)
}

int main() {
    char *s = "hello, world!";
    foo((void *)s);
    return 0;
}

```