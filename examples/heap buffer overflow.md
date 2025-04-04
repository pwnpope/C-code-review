
- A heap overflow condition is a buffer overflow, where the buffer that can be overwritten is allocated in the heap portion of memory, generally meaning that the buffer was allocated using a routine such as malloc().

# 2 Examples

```c
#define BUFSIZE 256  
int main(int argc, char **argv) {

char *buf;  
buf = (char *)malloc(sizeof(char)*BUFSIZE);

strcpy(buf, argv[1]);
}
```
- The buffer is allocated heap memory with a fixed size, but there is no guarantee the string in argv[1] will not exceed this size and cause an overflow.

---
- This example applies an encoding procedure to an input string and stores it into a buffer.
```c
char * copy_input(char *user_supplied_string){

	int i, dst_index;  
	char *dst_buf = (char*)malloc(4*sizeof(char) * MAX_SIZE);  

	if ( MAX_SIZE <= strlen(user_supplied_string) ){
		die("user string too long, die evil hacker!");
	}  
	dst_index = 0;  
	
	for ( i = 0; i < strlen(user_supplied_string); i++ ){
		if('&' == user_supplied_string[i] ){
			dst_buf[dst_index++] = '&';  
			dst_buf[dst_index++] = 'a';  
			dst_buf[dst_index++] = 'm';  
			dst_buf[dst_index++] = 'p';  
			dst_buf[dst_index++] = ';';
		}  
		else if ('<' == user_supplied_string[i] ){
	/* encode to &lt; */
	}  
		else dst_buf[dst_index++] = user_supplied_string[i];
	}  
	return dst_buf;
}
```
- The programmer attempts to encode the ampersand character in the user-controlled string, however the length of the string is validated before the encoding procedure is applied.
- Furthermore, the programmer assumes encoding expansion will only expand a given character by a factor of 4, while the encoding of the ampersand expands by 5. 
- As a result, when the encoding procedure expands the string it is possible to overflow the destination buffer if the attacker provides a string of many ampersands.