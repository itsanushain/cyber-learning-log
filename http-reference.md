HTTP for Pentesters — My Reference Notes

Personal notes from TryHackMe "HTTP in Detail" + PortSwigger labs. Written in my own words 

 What HTTP actually is
 
HyperText Transfer Protocol(HTTP) is the language a browser and a web server use to talk to each other.It's a request-reply protocol — the client sends a request,
the server sends back a response, and that's the entire conversation.The server only replies to what was asked ,Like a restaurant you (the client/browser)
call the waiter (the server) and ask for something — "bring me the menu,".The waiter never walks up to your table and give you the menu,not the food you did'nt ask for.
He only responds when you ask.Something that confused me at first was"Stateless" . It's like it doesn't "remember" that you logged in two requests ago. Each request
has to carry everything the server needs to understand it on its own — that's why cookies/tokens exist,because the protocol itself has no built-in memory.About encryption
HTTPS handles it.HTTP wrapped in TLS. When Burp intercepts "HTTPS traffic" it's actually terminating the TLS connection and showing me the plain HTTP underneath.
*HTTP--PORT 80
*HTTPS--PORT 443

Anatomy of a Request
   
Every request has three parts, always in this order: a start line, headers, then (optionally) a body
1. Method line
- GET /profile?user=daniel HTTP/1.1
Three pieces: the METHOD (what I want to do), the PATH + query string (what resource, and any parameters), and the HTTP VERSION. This one line tells the server everything about intent before it even reads a single header.
2. Headers
>Host — tells the server which website you want, since one server can host many.
>User-Agent — says what browser/device you're using (but it's easy to lie about).
>Cookie — sends your saved session ID back so the server remembers you're logged in.
>Content-Type — tells the server what format the data you're sending is in (JSON, form data, etc.).
>Content-Length — tells the server how many bytes of data to expect in the body.
>Authorization — carries your login credentials or access token.
>X-Forwarded-For — shows the original visitor's IP address when a proxy is in the middle.
3. Body
The body is where the actual "stuff" you're sending lives — like the text you typed in a form, a JSON object, or a file you're uploading. Not every request has one. A simple GET (just asking for a page) usually doesn't need a body — but POST, PUT, and PATCH (things that send data to the server) usually do.

Anatomy of a Response

 1. Status line
     HTTP/1.1 200 OK
HTTP version, a numeric status code, and a short human-readable reason phrase. I care about the number, not the words next to it — servers can put whatever text they want there.
 2.  Headers
     Set-Cookie — the server's way of saying "save this cookie and send it back to me next time."
     Content-Type — tells your browser what kind of content it's receiving (HTML page, JSON           data, image, etc.).
     Location — used with redirects to say "go to this URL instead."
     Cache-Control — tells the browser whether it's okay to save a copy of this response for          later.
     Server / X-Powered-By — reveals what software/tech the server is running on.
     Access-Control-Allow-Origin — says who else (which websites) is allowed to read this             response. 
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
