Detailed notes on *8. Authentication and authorization for backend engineers*

### *1. Core Definitions*
*   *Authentication (The "Who"):* The mechanism used to assign an identity to a subject. It answers the question, *"Who are you in a given context?"*.
*   *Authorization (The "What"):* The process of determining a user's permissions and capabilities. It answers the question, *"What can you do in this context?"*.

### *2. The Evolution of Authentication*
Understanding how authentication evolved helps explain modern systems:
*   *Pre-Industrial (Implicit Trust):* Relied on human contextual trust, like handshakes or a village elder vouching for someone.
*   *Medieval Era (Explicit Authentication):* Used wax seals as physical tokens of identity based on *possession*. This era saw the first "bypass attacks" via forgery.
*   *Industrial Revolution:* Telegraph operators used pre-agreed passphrases, shifting the paradigm to *something you know*.
*   *1960s (Mainframes):* MIT introduced passwords for multi-user systems. Initially stored in plain text, an incident where the password file was printed led to the invention of *hashing* (transforming passwords into secure, irreversible, fixed-length strings).
*   *1970s:* The invention of Diffie-Hellman introduced asymmetric cryptography, forming the backbone of modern Public Key Infrastructure (PKI) and early ticket-based authentication (Kerberos).
*   *1990s (MFA):* To combat brute force attacks, Multi-Factor Authentication (MFA) was introduced, combining three principles: something you know (password), something you have (OTP), and something you are (biometrics).
*   *Future Trends:* Post-Quantum Cryptography (to secure data against quantum computers), decentralized identity (blockchain), and behavioral biometrics.

### *3. Three Core Components of Modern Authentication*
#### *A. Sessions*
*   *The Problem:* HTTP is fundamentally a stateless protocol, meaning it remembers nothing about past requests. Modern web apps (like e-commerce sites) need stateful memory.
*   *The Solution:* The server creates a unique *Session ID* upon login, storing the user's data alongside this ID in a persistent, fast in-memory store like Redis. The Session ID is sent to the client and included in all future requests, giving the server memory.

#### *B. JWTs (JSON Web Tokens)*
*   *The Problem:* As apps scaled globally, storing and synchronizing millions of sessions across distributed servers caused latency and high storage costs.
*   *The Solution:* JWTs are a *stateless* mechanism that offloads storage from the server. 
*   *Structure:* They contain three base64-encoded parts:
    1.  *Header:* Metadata like the signing algorithm.
    2.  *Payload:* Contains "claims" such as the user ID (`sub`), issued at time (`iat`), and roles.
    3.  *Signature:* Verified using a secret key held only by the server to ensure the token hasn't been tampered with.

#### *C. Cookies*
*   *Definition:* A mechanism allowing servers to store small pieces of information directly in the user's browser.
*   *Workflow:* The server sets an `HTTP-only` cookie (so JavaScript cannot access it) containing the Auth Token. The browser then automatically attaches this cookie to every subsequent request sent to that specific server.

### *4. Major Types of Authentication*
Backend engineers typically use four main workflows:

1.  *Stateful Authentication:*
    *   *Workflow:* User logs in $\rightarrow$ Server stores session in Redis $\rightarrow$ Server sends Session ID via cookie $\rightarrow$ Client sends cookie on next request $\rightarrow$ Server looks up ID in Redis.
    *   *Pros/Cons:* Offers centralized control and easy token revocation, but has limited scalability and higher operational complexity. Ideal for standard web applications.
2.  *Stateless Authentication:*
    *   *Workflow:* User logs in $\rightarrow$ Server signs a JWT with a secret key $\rightarrow$ Client sends JWT in the `Authorization` header on next request $\rightarrow$ Server cryptographically verifies the token without a database lookup.
    *   *Pros/Cons:* Highly scalable and portable, but revoking access before the token expires is extremely complex without forcing all users to log out by changing the secret key.
3.  *API Key-Based Authentication:*
    *   *Purpose:* Designed for programmatic, *machine-to-machine* communication (e.g., your backend server requesting data from OpenAI's server).
    *   *Workflow:* A user generates a cryptographically random string via a UI. Their server then includes this key in automated requests to bypass human visual interactions (like login forms).
4.  *OAuth 2.0 & OpenID Connect (OIDC):*
    *   *The Delegation Problem:* Users historically shared passwords so one app could access resources on another (e.g., a travel app scanning Gmail). This was a massive security risk and made revocation impossible.
    *   *OAuth 2.0:* Solved this by allowing a client app to request a specific *token* with limited permissions (e.g., read-only access to contacts) from an Authorization Server. This handles *authorization*.
    *   *OpenID Connect (OIDC):* Because OAuth didn't verify identity*, OIDC was built on top. It introduced the **ID Token* (a JWT), which securely shares user profile data (like email and name). This powers the "Sign in with Google" features we use today.

### *5. Authorization: Role-Based Access Control (RBAC)*
*   *The Concept:* Not all users should have the same capabilities (e.g., standard users vs. admins accessing a "dead zone" of deleted files). 
*   *How RBAC Works:* Users are assigned specific roles (User, Admin, Moderator). Each role maps to strict resource permissions (Read, Write, Delete).
*   *Execution:* During the request cycle, the server deduces the user's role from their token. If an unauthorized user attempts an action, the server rejects it with a `403 Forbidden` status.

### *6. Critical Security Best Practices*
*   *Use Generic Error Messages:* Never respond with "User not found" or "Incorrect password." Attackers use these friendly messages to deduce valid usernames and increase their attack surface. Always use a generic response like *"Authentication failed"*.
*   *Prevent Timing Attacks:* Attackers can measure the time it takes for a server to respond. If a username is invalid, the server fails instantly. If the username is valid but the password is wrong, the server takes longer because it has to run the heavy password-hashing algorithm. 
    *   Solution: Backend engineers must equalize response times using *constant-time operations* or by *simulating a fake response delay* (e.g., a standard 200ms delay) so attackers cannot measure the difference.