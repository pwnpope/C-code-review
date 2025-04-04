- A double-free can take a bit of time to understand, but ultimately it is very simple.

- Firstly, remember that for fast chunks in the fastbin, the location of the next chunk in the bin is specified by the `fd` pointer. 
	- This means if chunk `a` points to chunk `b`, once chunk `a` is freed the next chunk in the bin is chunk `b`.

- In a double-free, we attempt to **control** `fd`. By overwriting it with an arbitrary memory address, we can tell `malloc()` _where the next chunk is to be allocated_. 
	- For example, say we overwrote `a->fd` to point at `0x12345678`; once `a` is free, the _next chunk on the list_ will be `0x12345678`.

- double free super basic example:
```c
char *a = malloc(0x20);
free(a);  /* free'd once: OK */
free(a);  /* free'd twice: BIG NONO */
```

- But what happens if we called `malloc()` again for the same size?
```c
char *b = malloc(0x20);
```

- Well, strange things would happen. 
	- `a` is both allocated (in the form of `b`) _and free at the same time_.

- If you remember, the heap attempts to save as much space as possible and when the chunk is free the `fd` pointer is `written where the user data used to be`.

![alt text](https://1919401647-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MEwBGnjPgf263kl5vWP%2F-MK9TNWL7biIXc0-fwXW%2F-MK9Xit_NuWfjHavDbpF%2Fimage.png?alt=media&token=b0847a19-9fb2-4e69-b8ba-08dc01964a41)

But what does this mean?
- When we write into the use data of `b`, we're writing into the `fd` of `a` _at the same time_.
	- And remember - controlling `fd` means we can control where the next chunk gets allocated! 
		- So we can `write an address` into the data of `b`, and that's where the next chunk gets placed.

```c
strcpy(b, "\x78\x56\x34\x12");
```
- Now, the next alloc will return `a` **again**. This doesn't matter, we want the one afterwards.

```c
malloc(0x20) /* This is yet another 'a', we can ignore this */
char *controlled = malloc(0x20); /* This is in the location we want */
```
- Boom - an arbitrary write.