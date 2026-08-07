HTTP for Pentesters — My Reference Notes

Personal notes from TryHackMe "HTTP in Detail" + PortSwigger labs. Written in my own words 

1. What HTTP actually is
HyperText Transfer Protocol(HTTP) is the language a browser and a web server use to talk to each other.It's a request-reply protocol — the client sends a request,
the server sends back a response, and that's the entire conversation.The server never only replies to what was asked ,Like a restaurant you (the client/browser)
call the waiter (the server) and ask for something — "bring me the menu,".The waiter never walks up to your table randomly and hands you food you didn't ask for.
He only responds when you ask.Something that confused me at first was"Stateless" . It's like it doesn't "remember" that you logged in two requests ago. Each request
has to carry everything the server needs to understand it on its own — that's why cookies/tokens exist,because the protocol itself has no built-in memory.About encryption
HTTPS handles it.HTTP wrapped in TLS. When Burp intercepts "HTTPS traffic" it's actually terminating the TLS connection and showing me the plain HTTP underneath.
*HTTP--PORT 80
*HTTPS--PORT 443

2. Anatomy of a Request
Every request has three parts, always in this order: a start line, headers, then (optionally) a body
- Method line —
- Headers (list the ones you actually understand well) —
- Body —

## 3. Anatomy of a Response
- Status line —
- Headers —
- Body —

## 4. HTTP Methods
| Method | What it does | Security relevance |
|--------|--------------|---------------------|
| GET    |              |                     |
| POST   |              |                     |
| PUT    |              |                     |
| DELETE |              |                     |
| OPTIONS|              |                     |
| HEAD   |              |                     |

## 5. Status Code Categories
| Range | Meaning | Example I've seen |
|-------|---------|--------------------|
| 1xx   |         |                    |
| 2xx   |         |                    |
| 3xx   |         |                    |
| 4xx   |         |                    |
| 5xx   |         |                    |

## 6. Cookies & Sessions
(How does a stateless protocol maintain login state? Explain the mechanism.)

## 7. How this connects to Burp Suite
(What am I actually looking at when I open Repeater/Intruder? Map the theory to the tool.)

## 8. Things I found confusing at first (and how I resolved them)
(This section is gold for interviews — shows real learning process, not just facts.)

## References
- TryHackMe: HTTP in Detail
- PortSwigger Web Security Academy
