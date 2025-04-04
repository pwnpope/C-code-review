
```c
char *b = malloc(0x256);
free(b);

printf(b); // use after free triggered
```
- anytime the variable is used after being freed is a UAF (use-after-free)