1. **Dangling Pointers**:
    - **Vulnerability**: Use-after-free is a common vulnerability associated with dangling pointers. After the memory has been freed, if the program continues to use the dangling pointer, it may lead to arbitrary code execution, denial of service, or information leakage, as the program might read from or write to memory areas that it doesn't have legitimate access to.

---

2. **Null Pointers**:
    - **Vulnerability**: Null pointer dereference is the vulnerability associated here. When a program tries to read or write to a location pointed to by a null pointer, it typically results in a runtime error or crash. In some cases, if an attacker can control or predict the actions following a null pointer dereference, they might exploit this behaviour for malicious purposes.

---

3. **Wild Pointers**:
    - **Vulnerability**: Wild pointers are similar to dangling pointers in their potential for causing vulnerabilities. Because they point to arbitrary locations, using them without proper initialization can lead to unpredictable behaviour, memory corruption, or sensitive data exposure. Accessing memory through wild pointers can result in security vulnerabilities such as buffer overflows or arbitrary code execution.

