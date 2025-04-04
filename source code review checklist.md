- list of rules including descriptions and in depth explanations: https://rules.sonarsource.com
	- including different languages.

---

#### Input Validation
- [ ]  Are inputs from external sources validated?
- [ ]  Is user input tested for type, length, format, and range, and by enforcing limits?
- [ ]  Are flaws in regular expressions causing data validation problems?
- [ ]  Are exact match approaches used?
- [ ]  Are allow list approaches used (i.e., check strings for only expected values)?
- [ ]  Are block list approaches used (i.e., rejected stings for inappropriate values)?
- [ ]  Are XML documents validated against their schemas?
- [ ]  Are string concatenations NOT used for user input?
- [ ]  Are SQL statements NOT dynamically created by using user input?
- [ ]  Is data validated on the server side?
- [ ]  Is there a strong separation between data and commands, and data and client-side scripts?
- [ ]  Is contextual escaping used when passing data to SQL, LDAP, OS and third-party commands?
- [ ]  Are https headers validated for each request?

#### Authentication and User Management
- [ ]  Are sessions handled correctly?
- [ ]  Do failure messages for invalid usernames or passwords NOT leak information?
- [ ]  Are invalid passwords NOT logged (which can leak sensitive password & user name combinations)?
- [ ]  Are the password requirements (lengths/complexity) appropriate?
- [ ]  Are invalid login attempts correctly handled with lockouts, and rate limits?
- [ ]  Does the “forgot password” routine NOT leak information, and is NOT vulnerable to spamming?
- [ ]  Are passwords NOT sent in plain text via email?
- [ ]  Are appropriate mechanisms such as hashing, salts, and encryption used for storing passwords and usernames?

#### Authorization
- [ ]  Are authentication and authorization the first logic executed for each request?
- [ ]  Are authorization checks granular (page and directory level)?
- [ ]  Is access to pages and data denied by default?
- [ ]  Is re-authenticate for requests that have side effects enforced?
- [ ]  Are there clear roles for authorization?
- [ ]  Can authorization NOT be circumvented by parameter or cookie manipulation?

#### Session Management
- [ ]  Are session parameters NOT passed in URLs?
- [ ]  Do session cookies expire in a reasonably short time?
- [ ]  Are session cookies encrypted?
- [ ]  Is session data being validated?
- [ ]  Is private data in cookies kept to a minimum?
- [ ]  Does the application avoid excessive cookie use?
- [ ]  Is the session id complex?
- [ ]  Is the session storage secure?
- [ ]  Does the application properly handle invalid session ids?
- [ ]  Are session limits e.g., inactivity timeouts, enforced?
- [ ]  Are logouts invalidating the session?
- [ ]  Are session resources released when sessions are invalidated?

#### Encryption and Cryptography
- [ ]  Are state-of-the-art encryption algorithms used?
- [ ]  Are minimum key sizes supported?
- [ ]  What types of data must be encrypted?
- [ ]  Has sensitive data been secured in memory, storage and transit?
- [ ]  Do restricted areas require SSL/TLS?
- [ ]  Is sensitive information passed to/from non-SSL pages?

#### Exception Handling
- [ ]  Do all methods have appropriate exceptions?
- [ ]  Do error messages shown to users NOT reveal sensitive information including stack traces, or ids?
- [ ]  Does the application fail securely when exceptions occur?
- [ ]  Are system errors NOT shown to users?
- [ ]  Are resources released and transactions rolled back when there is an error?
- [ ]  Are all user or system actions are logged?
- [ ]  Do we make sure that sensitive information is NOT logged (e.g. passwords)?
- [ ]  Do we make sure we have logs or all important user management events (e.g. password reset)?
- [ ]  Are unusual activities such as multiple login attempts logged?
- [ ]  Do logs have enough detail to reconstruct events for audit purposes?