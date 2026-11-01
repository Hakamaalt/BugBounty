# Day 01 — Getting Started with Burp Suite & Web Security

**Author:** Harsh Dwivedi
**Learning Track:** Bug Bounty Hunting  
**Topics:** Burp Suite • HTTP Fundamentals • Proxy Configuration • Sessions • Authorization Testing

> Personal learning notes from my first practical session on web application security. The focus is on understanding how browser requests communicate with servers and how Burp Suite helps analyze application behavior. All practical testing should be performed on authorized targets or training labs.

---

## Learning Goals

By the end of this session, I wanted to understand:

- The basic working environment of Burp Suite.
- How HTTP requests and responses travel between browser and server.
- The difference between GET and POST methods.
- How browser cookies and authentication sessions work.
- How to connect Firefox with Burp Suite using FoxyProxy.
- How to read HTTP status codes.
- The basics of identifying access control weaknesses.

## 1. Introduction to Burp Suite

Burp Suite is a web application security testing toolkit that allows security researchers to observe and analyze communication between a browser and a web server.

During a normal browsing session, requests are sent directly to the server. When a proxy is configured, these requests can be inspected before they reach their destination.

### Important Burp Suite Modules

| Module | What I Learned |
|---|---|
| Proxy | Captures browser requests and server responses |
| HTTP History | Stores previously captured HTTP traffic |
| Repeater | Allows manual modification and resending of requests |
| Intruder | Helps perform controlled automated request testing |
| Decoder | Converts encoded data into readable formats |
| Comparer | Highlights differences between two HTTP messages |
| Sequencer | Analyzes randomness in tokens |
| Target | Organizes application hosts and discovered paths |

### My First Burp Suite Workflow

1. Launch Burp Suite and create a temporary project.
2. Open the Proxy section.
3. Check that the proxy listener is active.
4. Configure Firefox to communicate through the local proxy.
5. Open an authorized testing application.
6. Browse different pages and generate HTTP traffic.
7. Visit HTTP History to inspect captured requests.
8. Send selected requests to Repeater for manual analysis.

**Observation:** Burp Suite makes it easier to understand what happens behind the browser interface.

---

## 2. Connecting Firefox with Burp Suite Using FoxyProxy

FoxyProxy is a Firefox extension that helps switch between different proxy configurations.

I learned how to route browser traffic through Burp Suite using a local HTTP proxy.

### Proxy Configuration

| Setting | Configuration |
|---|---|
| Proxy Type | HTTP |
| IP Address | `127.0.0.1` |
| Port | `8080` |
| Browser | Firefox |
| Extension | FoxyProxy |

### Configuration Steps

1. Open Firefox.
2. Visit the browser extension store.
3. Install FoxyProxy.
4. Open the FoxyProxy settings.
5. Add a new proxy configuration.
6. Select HTTP as the proxy type.
7. Enter `127.0.0.1` in the hostname field.
8. Set the port to `8080`.
9. Save the configuration.
10. Start Burp Suite and enable the corresponding proxy listener.
11. Activate the FoxyProxy profile in Firefox.
12. Visit an authorized test website and check Burp's HTTP History.

For HTTPS traffic inspection, configure Burp's CA certificate in a dedicated testing browser profile using the official documentation.

### Things to Check if Proxy Is Not Working

- Is Burp Suite running?
- Is the proxy listener enabled?
- Does FoxyProxy use the correct IP and port?
- Is the correct browser profile active?
- Has the Burp CA certificate been configured for HTTPS inspection?

**My Observation:** The browser can now send its traffic through Burp Suite, allowing me to inspect the communication between the application and server.

---

## 3. Understanding HTTP Requests

HTTP stands for Hypertext Transfer Protocol. It defines how clients and servers exchange information.

Whenever I open a website, submit a form or access an API, HTTP requests and responses are involved.

### Main Parts of an HTTP Request

- **Method:** Describes the requested operation.
- **Path:** Identifies the requested resource.
- **Headers:** Carry additional request information.
- **Body:** Contains submitted data when applicable.
- **Cookies:** May carry session identifiers.

### Example Request

```http
GET /dashboard HTTP/1.1
Host: app.example.test
Accept: text/html
Cookie: session=[REDACTED]
```

In this example:

- `GET` is the request method.
- `/dashboard` is the requested path.
- `Host` identifies the destination.
- `Cookie` contains session information.

This is important because Burp Suite allows us to inspect these individual components.

---

## 4. GET vs POST Methods

Two commonly encountered HTTP methods are GET and POST.

| Feature | GET | POST |
|---|---|---|
| Common purpose | Retrieve information | Submit information |
| Parameters | Often in URL | Often in request body |
| Request body | Usually absent | Commonly present |
| HTTP safe semantics | Yes | No |
| Idempotent semantics | Yes | Not guaranteed |
| Common example | Loading a profile | Submitting a login form |

### GET Example

```http
GET /user/profile?id=42 HTTP/1.1
Host: app.example.test
Accept: application/json
```

### POST Example

```http
POST /api/profile HTTP/1.1
Host: app.example.test
Content-Type: application/json
Cookie: session=[REDACTED]

{
  "displayName": "Test User"
}
```

**Security Note:** Using POST does not automatically make a request secure. Both methods require proper authentication, authorization and input validation.

---

## 5. Cookies, Authentication Tokens & Sessions

Understanding session management is important when testing web applications.

### Cookies

Cookies are small pieces of data stored by the browser and sent to matching websites according to their scope and security rules.

Some important cookie attributes:

| Attribute | Purpose |
|---|---|
| Secure | Restricts cookie transmission to HTTPS |
| HttpOnly | Prevents normal JavaScript access |
| SameSite | Controls certain cross-site cookie requests |
| Domain | Defines the cookie's domain scope |
| Path | Defines the applicable URL path |
| Expires | Defines the expiration time |

### Authentication Tokens

Tokens are credentials used by applications to identify or authorize users.

Examples include:

- Bearer access tokens
- JWT tokens
- API authentication tokens

Important things to remember:

- Never expose tokens publicly.
- Treat tokens like passwords.
- JWT payloads are generally encoded, not encrypted by default.
- Token signatures, expiration and permissions must be validated correctly.

### Session Management

A session allows an application to remember an authenticated user across multiple requests.

Things worth understanding during authorized testing:

- Does the session change after login?
- Does logout invalidate the session?
- Are cookies configured securely?
- Does password reset affect existing sessions?
- Does the server check authorization on protected requests?

---

## 6. Understanding HTTP Status Codes

While inspecting Burp Suite responses, status codes help understand how the server handled a request.

| Status Code | Meaning |
|---|---|
| `200 OK` | Request processed successfully |
| `201 Created` | A resource was created |
| `204 No Content` | Successful response without a body |
| `301 Moved Permanently` | Permanent redirect |
| `302 Found` | Temporary redirect |
| `400 Bad Request` | Invalid request |
| `401 Unauthorized` | Authentication required or invalid |
| `403 Forbidden` | Access refused |
| `404 Not Found` | Resource not found or concealed |
| `405 Method Not Allowed` | HTTP method not supported |
| `409 Conflict` | Conflict with current resource state |
| `415 Unsupported Media Type` | Unsupported content type |
| `429 Too Many Requests` | Rate limit reached |
| `500 Internal Server Error` | Server-side error |

**Important Learning:** A status code by itself does not establish a vulnerability. The response body, session state and actual application behavior must also be examined.

---

## 7. Introduction to Authorization Testing

Authentication and authorization are different concepts.

- **Authentication:** Determines who the user is.
- **Authorization:** Determines what the user is allowed to access or perform.

An application can authenticate a user correctly but still contain authorization weaknesses.

### Common Access Control Issues

- IDOR (Insecure Direct Object Reference)
- BOLA (Broken Object Level Authorization)
- BFLA (Broken Function Level Authorization)
- Horizontal privilege escalation
- Vertical privilege escalation
- Unauthorized access to restricted functionality

### Basic Testing Approach

For a controlled lab, use two accounts that belong to you.

1. Create Account A and Account B.
2. Create harmless test objects under each account.
3. Log in as Account A.
4. Capture a normal request for an object owned by Account A.
5. Record the endpoint and object reference.
6. Switch to Account B using a separate test session.
7. Compare the normal behavior of both accounts.
8. Check whether the server enforces object ownership.
9. Document expected and actual behavior.

Do not access other users' information or enumerate real production identifiers.

### What I Should Observe

- Can one test account access another test account's object?
- Does the server enforce role permissions?
- Are restricted actions protected on the backend?
- Does hiding a button in the interface actually prevent the corresponding server operation?

---

## 8. Practical Learning Checklist

Use this checklist to track what I completed during the session.

- [ ] Opened Burp Suite successfully.
- [ ] Understood the purpose of Proxy.
- [ ] Explored HTTP History.
- [ ] Learned the basic use of Repeater.
- [ ] Installed FoxyProxy in Firefox.
- [ ] Configured `127.0.0.1:8080`.
- [ ] Captured browser traffic.
- [ ] Understood HTTP request structure.
- [ ] Compared GET and POST.
- [ ] Learned about cookies and authentication tokens.
- [ ] Understood session management basics.
- [ ] Reviewed common HTTP status codes.
- [ ] Learned the difference between authentication and authorization.
- [ ] Understood the basics of access control testing.

---

## 9. Key Takeaways

After completing this session, these are the main concepts I want to remember:

1. Burp Suite is useful for inspecting and analyzing web application traffic.
2. FoxyProxy helps connect Firefox to Burp Suite.
3. HTTP requests contain methods, paths, headers and sometimes a body.
4. GET and POST serve different purposes but neither provides authorization by itself.
5. Cookies and tokens must be treated as sensitive credentials.
6. HTTP status codes need to be interpreted in application context.
7. Authentication and authorization are separate security concepts.
8. Access control must be enforced by the server, not just the browser interface.

---

## 10. Learning Resources

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MDN HTTP Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP)

---


**Session Status:** Day 01 Completed

