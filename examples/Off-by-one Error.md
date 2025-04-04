- A product calculates or uses an incorrect maximum or minimum value that is 1 more, or 1 less, than the correct value.

- This weakness will generally lead to undefined behavior and therefore crashes. 
	- In the case of overflows involving loop index variables, the likelihood of infinite loops is also high.

- This weakness can sometimes trigger buffer overflows which can be used to execute arbitrary code. This is usually outside the scope of a program's implicit security policy.

---
# 4 Examples

- The following code allocates memory for a maximum number of widgets.
- It then gets a user-specified number of widgets, making sure that the user does not request too many.
- It then initializes the elements of the array using InitializeWidget().
- Because the number of widgets can vary for each request, the code inserts a NULL pointer to signify the location of the last widget.

```c
int i;  
unsigned int numWidgets;  
Widget **WidgetList;  
  
numWidgets = GetUntrustedSizeValue();  
if ((numWidgets == 0) || (numWidgets > MAX_NUM_WIDGETS)) {

ExitError("Incorrect number of widgets requested!");

}  
WidgetList = (Widget **)malloc(numWidgets * sizeof(Widget *));  
printf("WidgetList ptr=%p\n", WidgetList);  
for(i=0; i<numWidgets; i++) {

WidgetList[i] = InitializeWidget();

}  
WidgetList[numWidgets] = NULL;  
showWidgets(WidgetList);
```
- However, this code contains an off-by-one calculation error ([CWE-193](https://cwe.mitre.org/data/definitions/193.html)). 
- It allocates exactly enough space to contain the specified number of widgets, but it does not include the space for the NULL pointer. 
- As a result, the allocated buffer is smaller than it is supposed to be ([CWE-131](https://cwe.mitre.org/data/definitions/131.html)). So if the user ever requests MAX_NUM_WIDGETS, there is an out-of-bounds write ([CWE-787](https://cwe.mitre.org/data/definitions/787.html)) when the NULL is assigned.
- Depending on the environment and compilation settings, this could cause memory corruption.

---

- The first call to strncat() appends up to 20 characters plus a terminating null character to fullname[].
- There is plenty of allocated space for this, and there is no weakness associated with this first call. 
- However, the second call to strncat() potentially appends another 20 characters. The code does not account for the terminating null character that is automatically added by strncat(). 
- This terminating null character would be written one byte beyond the end of the fullname[] buffer.
- Therefore an off-by-one error exists with the second strncat() call, as the third argument should be 19.

```c
char firstname[20];  
char lastname[20];  
char fullname[40];  
  
fullname[0] = '\0';  
  
strncat(fullname, firstname, 20);  
strncat(fullname, lastname, 20);
```
- When using a function like strncat() one must leave a free byte at the end of the buffer for a terminating null character, thus avoiding the off-by-one weakness. 
- Additionally, the last argument to strncat() is the number of characters to append, which must be less than the remaining space in the buffer.
- Be careful not to just use the total size of the buffer.

---
- The Off-by-one error can also be manifested when reading characters from a character array within a for loop that has an incorrect continuation condition.
```c
#define PATH_SIZE 60  
  
char filename[PATH_SIZE];  
  
for(i=0; i<=PATH_SIZE; i++) {  
	char c = getc();  
	
if (c == 'EOF') { filename[i] = '\0'; } 

filename[i] = getc();
}
```

---
- As another example the Off-by-one error can occur when using the sprintf library function to copy a string variable to a formatted string variable and the original string variable comes from an untrusted source. 
- As in the following example where a local function, setFilename is used to store the value of a filename to a database but first uses sprintf to format the filename.
- The setFilename function includes an input parameter with the name of the file that is used as the copy source in the sprintf function. 	
- The sprintf function will copy the file name to a char array of size 20 and specifies the format of the new variable as 16 characters followed by the file extension .dat.
```c
int setFilename(char *filename) {

char name[20];  
sprintf(name, "%16s.dat", filename);  
int success = saveFormattedFilenameToDB(name);  
return success;
}
```
- However this will cause an Off-by-one error if the original filename is exactly 16 characters or larger because the format of 16 characters with the file extension is exactly 20 characters and does not take into account the required null terminator that will be placed at the end of the string.

---
## Mitigations:
- When copying character arrays or using character manipulation methods, the correct size parameter must be used to account for the null terminator that needs to be added at the end of the array. 

- Some examples of functions susceptible to this weakness in C include strcpy(), strncpy(), strcat(), strncat(), printf(), sprintf(), scanf() and sscanf().