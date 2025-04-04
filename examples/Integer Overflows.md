## Integer Overflow

- Since an integer is a fixed size (32 bits for the purposes of this paper), there is a fixed maximum value it can store. 

-  When an attempt is made to `store a value greater than this maximum value` it is known as an integer overflow.

- Integer overflows cannot be detected after they have happened, so there is no way for an application to tell if a result it has calculated previously is in fact correct.

- Most integer overflows are not exploitable because memory is not being directly overwritten, but `sometimes they can lead to other classes of bugs`, frequently buffer overflows. integer overflows can be difficult to spot, so even well audited code can spring surprises.

---
## Signedness Bugs
- Signedness bugs occur when an `unsigned variable is interpreted as signed`,
	or when a signed variable is interpreted as unsigned.

- Signedness bugs can take a variety of forms, but some of the things to look
out for are:

	* signed integers being used in comparisons
	* signed integers being used in arithmetic
	* unsigned integers being compared to signed integers

- example:
```c
int copy_something(char *buf, int len){
	char kbuf[800];
	if(len > sizeof(kbuf)){         /* [1] */
		return -1;
    }
    return memcpy(kbuf, buf, len);  /* [2] */
}
```

- The problem here is that `memcpy takes an unsigned int as the len parameter`, but the bounds check performed before the memcpy is done using signed integers.

- By passing a negative value for len, it is possible to pass the check at [1], but then in the call to memcpy at [2], len will be interpreted as a huge unsigned value, causing `memory to be overwritten well past theend of the buffer kbuf`.

- Another problem that can stem from signed/unsigned confusion occurs when
arithmetic is performed.  Consider the following example:
```c
int table[800];

int insert_in_table(int val, int pos){
    if(pos > sizeof(table) / sizeof(int)){
        return -1;
    }

    table[pos] = val;

    return 0;
}
```

Since the line
```c
table[pos] = val;
```
is equivalent to
```c
*(table + (pos * sizeof(int))) = val;
```

- we can see that the problem here is that the code does not expect a negative operand for the addition:
	- it expects (table + pos) to be greater than table, so providing a negative value for pos causes a situation which the program does not expect and can therefore not deal with.

#### Exploiting Signedness Bugs

- This class of bug can be problematic to exploit, due to the fact that signed integers, when interpreted as unsigned, tend to be huge. 
	- For example, -1 when represented in hexadecimal is 0xffffffff

- When interpreted as unsigned, this becomes the greatest value it is possible to
	represent in an integer (4,294,967,295), so if this value is passed to mempcpy as the len parameter (for example), memcpy will attempt to copy 4GB of data to the destination buffer. 
	- Obviously this is likely to cause a segfault or, if not, to trash a large amount of the stack or heap. Sometimes it is possible to get around this problem by passing a very low value for the source address and hope, but this is not always possible.

---
## Signedness bugs caused by integer overflows

- Sometimes, it is possible to overflow an integer so that it wraps around to a negative number. 
	- Since the application is unlikely to expect such a value, it may be possible to `trigger a signedness bug` as described above.

- example:
```c
  
int get_two_vars(int sock, char *out, int len){
    char buf1[512], buf2[512];
    unsigned int size1, size2;
    int size;

    if(recv(sock, buf1, sizeof(buf1), 0) < 0){
        return -1;
    }
    if(recv(sock, buf2, sizeof(buf2), 0) < 0){
        return -1;
    }

    /* packet begins with length information */
    memcpy(&size1, buf1, sizeof(int));
    memcpy(&size2, buf2, sizeof(int));

    size = size1 + size2;       /* [1] */

    if(size > len){             /* [2] */
        return -1;
    }

    memcpy(out, buf1, size1);
    memcpy(out + size1, buf2, size2);

    return size;
}
```

- This example shows what can sometimes happen in network daemons, especially
when length information is passed as part of the packet (in other words, it is supplied by an untrusted user).

- The addition at [1], used to check that the data does not exceed the bounds of the output buffer, can be abused by setting size1 and size2 to values that will cause the size variable to wrap around to a negative value.
	- example:
		- size1 = 0x7fffffff
	    - size2 = 0x7fffffff
	    - (0x7fffffff + 0x7fffffff = 0xfffffffe (-2)).

- When this happens, the bounds check at [2] passes, and a lot more of the out buffer can be written to than was intended (in fact, `arbitrary memory can be written to`, as the (out + size1) dest parameter in the second memcpy call allows us to get to any location in memory).

- These bugs can be exploited in exactly the same way as regular signedness bugs and have the same problems associated with them - i.e. negative values translate to huge positive values, which can easily cause segfaults.

---
## Widthness Overflow

- So an integer overflow is the result of attempting to store a value in a variable which is too small to hold it. 
	- The simplest example of this can be demonstrated by simply `assigning the contents of large variable to a smaller one`:
```c
#include <stdio.h>  


int main(void){  
	int l;  
    short s;  
    char c;  
  
    l = 0xdeadbeef;  
    s = l;  
    c = l;  
  
	printf("l = 0x%x (%d bits)\n", l, sizeof(l) * 8);  
    printf("s = 0x%x (%d bits)\n", s, sizeof(s) * 8);  
    printf("c = 0x%x (%d bits)\n", c, sizeof(c) * 8);  
  
    return 0;
}
```
- output:
```
l = 0xdeadbeef (32 bits)
s = 0xffffbeef (16 bits)
c = 0xffffffef (8 bits)
```

- Since each assignment causes the bounds of the values that can be stored in each type to be exceeded, the value is truncated so that it can fit in the variable it is assigned to.