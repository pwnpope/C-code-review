- Here are a couple examples from CTFs:


![](https://i.imgur.com/pOu1CbA.png)
- since our input `isn't properly null terminated` when it's printed out when we input r3dDr4g3nst1str0f1 plus more data to stuff the buffer, when it prints out our input back to us it contains a memory leak, this is our `out of bounds read`.

---

```c
         fwrite("\n[*] Insert new name (minimum 5 chars): ",1,0x28,stdout);
         fflush(stdout);
         num_bytes = read(0,buf,99);
         fflush(stdout);
         fprintf(stdout,"\n[*] Are you sure you want to use the name %s\n(y/n): ",buf);
```
- in this snippet from menu function, we can see that it is using read to read from stdin placing the data into a 264 char buffer and the read size is 99, due to the improper null termination, when the buffer is printed back to us there can be leaks.