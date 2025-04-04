```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char username[40];
    char user_role[15];
    int user_id;
} User;

User *create_user(const char *username, const char *role, int id) {
    User *new_user = (User *)malloc(sizeof(User));
    if (!new_user) {
        printf("Failed to allocate memory.\n");
        exit(1);
    }
    
    strncpy(new_user->username, username, sizeof(new_user->username) - 1);
    new_user->username[sizeof(new_user->username) - 1] = '\0';
    strncpy(new_user->user_role, role, sizeof(new_user->user_role) - 1);
    new_user->user_role[sizeof(new_user->user_role) - 1] = '\0';
    new_user->user_id = id;

    return new_user;
}

void change_role(User *user, const char *new_role) {
    if (user->user_id == 0) {
        printf("Cannot change role for this user.\n");
        return;
    }

    strncpy(user->user_role, new_role, sizeof(user->user_role));
}

void print_user_info(User *user) {
    printf("Username: %s\n", user->username);
    printf("User Role: %s\n", user->user_role);
    printf("User ID: %d\n", user->user_id);
}

int main() {
    User *admin = create_user("admin", "administrator", 0);
    User *guest = create_user("guest", "guest", 1);
    
    printf("Before role change:\n");
    print_user_info(admin);
    
    printf("\nChanging role...\n");
    change_role(admin, "root\0guest");
    
    printf("\nAfter role change:\n");
    print_user_info(admin);

    free(admin);
    free(guest);

    return 0;
}
```
- one byte overflow in change_role:
	- The strncpy in the change_role function does not account for the null terminator. If a string of exactly 15 characters (the size of user_role) is passed, it will overwrite the next field, user_id, due to the lack of a null terminator.


---

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NAME_LEN 50

typedef struct {
    char *name;
    int age;
} Person;

Person people[10];
int num_people = 0;

void add_person(char *name, int age) {
    if (num_people >= 10) {
        printf("Max people added.\n");
        return;
    }

    people[num_people].name = malloc(MAX_NAME_LEN);
    strcpy(people[num_people].name, name);
    people[num_people].age = age;
    
    num_people++;
}

void delete_person(char *name) {
    for (int i = 0; i < num_people; i++) {
        if (strcmp(people[i].name, name) == 0) {
            free(people[i].name);
            for (int j = i; j < num_people - 1; j++) {
                people[j] = people[j+1];
            }
            num_people--;
        }
    }
}

void print_people() {
    for (int i = 0; i < num_people; i++) {
        printf("Name: %s, Age: %d\n", people[i].name, people[i].age);
    }
}

int main() {
    add_person("Alice", 25);
    add_person("Bob", 30);
    
    print_people();
    
    delete_person("Alice");
    print_people();
    
    return 0;
}

```
- `Heap-based Buffer Overflow`: The function add_person allocates a fixed length of MAX_NAME_LEN for the name of the person. However, it does not check the length of the provided name before copying it using strcpy. 

---

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int secret_code;
    int user_permission;
} Credentials;

void grant_access(Credentials *cred) {
    if (cred->user_permission == 1) {
        printf("Access granted.\n");
    } else {
        printf("Access denied.\n");
    }
}

void enter_secret_code(Credentials *cred) {
    printf("Enter your secret code: ");
    scanf("%d", &(cred->secret_code));
    
    if (cred->secret_code == 12345) {
        cred->user_permission = 1;
    }
}

int main() {
    Credentials user = {0, 0};
    
    enter_secret_code(&user);
    grant_access(&user);
    
    return 0;
}
```

- The code contains a hardcoded secret (`12345`), which is a weak form of authentication. This kind of secret can be easily guessed, especially when it's a commonly known sequence. In a real-world scenario, an attacker might find this secret in various ways, such as by disassembling the program, analyzing memory dumps, or simply by brute-forcing.

---

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    char *note;
} Memo;

Memo *create_memo(const char *text) {
    Memo *new_memo = (Memo *)malloc(sizeof(Memo));
    if (!new_memo) {
        printf("Failed to allocate memory.\n");
        exit(1);
    }
    
    new_memo->note = (char *)malloc(strlen(text) + 1);
    strcpy(new_memo->note, text);
    
    return new_memo;
}

void delete_memo(Memo *memo) {
    free(memo->note);
    free(memo);
}

void update_memo(Memo *memo, const char *text) {
    memo->note = realloc(memo->note, strlen(text) + 1);
    strcpy(memo->note, text);
}

void print_memo(Memo *memo) {
    printf("Memo Note: %s\n", memo->note);
}

int main() {
    Memo *my_memo = create_memo("This is an initial note.");
    print_memo(my_memo);
    
    printf("\nUpdating memo...\n");
    update_memo(my_memo, "This is a much longer note. The previous memory allocation was not enough to store this new content.");
    print_memo(my_memo);

    delete_memo(my_memo);

    return 0;
}
```

- **Double Free**: The potential arises when `realloc` fails. We overwrite `memo->note` with the return value of `realloc` (which could be `NULL` if there's a failure). The original memory pointed to by `memo->note` is then lost (a memory leak) and on the subsequent call to `free(memo->note)` in `delete_memo`, it could result in a double free if `memo->note` is `NULL`.

- **Memory Leak**: As described above, if `realloc` fails, we lose the reference to the originally allocated memory, leading to a leak.

---

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_USERS 3

typedef struct {
    char username[50];
    int is_admin;
} User;

User users[MAX_USERS];

int authenticate(char* username, char* password) {
    if(strcmp(username, "admin") == 0 && strcmp(password, "supersecure") == 0) {
        return 1;
    }
    return 0;
}

void add_user() {
    for (int i = 0; i < MAX_USERS; i++) {
        if (users[i].username[0] == '\0') {
            printf("Enter username: ");
            scanf("%s", users[i].username);
            
            char pass[50];
            printf("Enter password: ");
            scanf("%s", pass);

            users[i].is_admin = authenticate(users[i].username, pass);
            printf("User added successfully.\n");
            return;
        }
    }
    printf("User list is full.\n");
}

void list_users() {
    for (int i = 0; i < MAX_USERS; i++) {
        if (users[i].username[0] != '\0') {
            printf("Username: %s, Is Admin: %d\n", users[i].username, users[i].is_admin);
        }
    }
}

int main() {
    memset(users, 0, sizeof(users));
    
    while (1) {
        printf("\n1. Add user\n");
        printf("2. List users\n");
        printf("3. Exit\n");

        int choice;
        printf("\nEnter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                add_user();
                break;
            case 2:
                list_users();
                break;
            case 3:
                exit(0);
            default:
                printf("Invalid choice.\n");
                break;
        }
    }

    return 0;
}
```
1. **scanf %s Buffer Overflow**: The `scanf("%s", ...)` pattern does not limit the number of characters read into a string, so it's vulnerable to buffer overflow. 
    
2. **Weak Credentials**: The hardcoded "`admin`" and "`supersecure`" credentials are a vulnerability because they're hard-coded directly into the program, making them discoverable to anyone who has access to the source or binary of the application.

---

```c
#include <stdio.h>
#include <pthread.h>

#define ITERATIONS 100000

long long int counter = 0;

void *increment(void *arg) {
    for (int i = 0; i < ITERATIONS; i++) {
        counter++;
    }
    return NULL;
}

void *decrement(void *arg) {
    for (int i = 0; i < ITERATIONS; i++) {
        counter--;
    }
    return NULL;
}

int main() {
    pthread_t tid1, tid2;

    pthread_create(&tid1, NULL, increment, NULL);
    pthread_create(&tid2, NULL, decrement, NULL);

    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    printf("Counter value: %lld\n", counter);

    return 0;
}
```

- **Race Condition**: This program has two threads one that increments a global counter by a certain number of iterations and another that decrements it by the same amount. If the threads worked perfectly in isolation, the final value of `counter` should be `0`. However, due to the race condition, the output is often non-zero because the two threads are contending to update the shared `counter` variable.
	- **Note**: For this code to compile, you might need to link against the pthread library: `gcc filename.c -lpthread`.

---

```c
#include <stdio.h>  
#include <string.h>  
  
int main() {  
   char buffer[10];  
   int i;  
  
   strcpy(buffer, "Hello");  
  
   for (i = 0; i < 5; i--) {  
       putchar(buffer[i]);  
   }  
  
   return 0;  
}
```

- **Out of Bounds Read**: the vulnerable line `for (i = 0; i < 5; i--)` will decrement the index counter reading before our buffer leaking memory until the program segfaults.