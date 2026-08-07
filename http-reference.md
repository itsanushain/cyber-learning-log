HTTP for Pentesters — My Reference Notes

Personal notes from TryHackMe "HTTP in Detail" + PortSwigger labs. Written in my own words 

 What HTTP actually is
 
HyperText Transfer Protocol(HTTP) is the language a browser and a web server use to talk to each other.It's a request-reply protocol — the client sends a request,
the server sends back a response, and that's the entire conversation.The server only replies to what was asked ,Like a restaurant you (the client/browser)
call the waiter (the server) and ask for something — "bring me the menu,".The waiter only walks up to your table and give you the menu,not the food you did'nt ask for.
He only responds when you ask.Something that confused me at first was"Stateless" . It's like it doesn't "remember" that you logged in two requests ago. Each request
has to carry everything the server needs to understand it on its own — that's why cookies/tokens exist,because the protocol itself has no built-in memory.About encryption
HTTPS handles it.HTTP wrapped in TLS. When Burp intercepts "HTTPS traffic" it's actually terminating the TLS connection and showing me the plain HTTP underneath.

*HTTP--PORT 80
*HTTPS--PORT 443

Anatomy of a Request
   
Every request has three parts, always in this order: a start line, headers, then (optionally) a body
1. Method line

#GET /profile?user=daniel HTTP/1.1

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
    
#HTTP/1.1 200 OK

HTTP version, a numeric status code, and a short human-readable reason phrase. I care about the number, not the words next to it — servers can put whatever text they want there.
 2. Headers
    
     Set-Cookie — the server's way of saying "save this cookie and send it back to me next time."
     Content-Type — tells your browser what kind of content it's receiving (HTML page, JSON form data, image, etc.).
     Location — used with redirects to say "go to this URL instead."
     Cache-Control — tells the browser whether it's okay to save a copy of this response for later.
     Server / X-Powered-By — reveals what software/tech the server is running on.
     Access-Control-Allow-Origin — says who else (which websites) is allowed to read this response. 
3. Body

The actual content — HTML, JSON, an image, a file download, whatever. This is what ends up rendered in the browser or parsed by whatever app made the request.

HTTP Methods

>GET - Asks for data, doesn't change anything.Data shows up in the URL — can leak in history/logs. Risky if used to trigger actions (CSRF).
>POST	- Sends new data to the server.	Main spot to test for injection attacks (SQLi, XSS) and login abuse.
>PUT	- Replaces/uploads a resource.	If open to anyone, attackers can overwrite or upload files.
>DELETE -	Removes a resource.	If not properly protected, anyone could delete things they shouldn't.
>OPTIONS	- Asks "what methods are allowed here?"	Quick way to check if risky methods (PUT/DELETE) are enabled.
>HEAD	- Same as GET, but no body — just headers.	Lets you check if something exists without downloading it — good for stealthy recon.


Status Code Categories

1xx	Informational — request received, continuing process.	
Example: 101 Switching Protocols, seen during a WebSocket upgrade handshake.
2xx	Success — the request was received, understood, and accepted.	
Example: 200 OK on a normal page load; 201 Created after registering an account.
3xx	Redirection — further action needed to complete the request.	
Example: 302 Found after a login redirecting me to /dashboard; useful to check for open redirects.
4xx	Client error — something's wrong with my request.
Example: 401 Unauthorized vs 403 Forbidden — I mix these up (see section 8); 404 Not Found while fuzzing directories.
5xx	Server error — the server failed while handling a valid request.
Example: 500 Internal Server Error when I break input parsing with a malformed payload — often leaks stack traces.

Cookies & Sessions

HTTP has no memory (section 1) — the server forgets you the second the response is sent. So how does a site keep you "logged in" as you click around? Cookies. Here's the step-by-step flow lets dig in

Step 1 — You log in
You type your username and password and hit submit. The server checks them against its database. Correct? It creates a session for you — basically a note in its own memory saying "username1 is logged in" — and generates a random, hard-to-guess session ID for that note, e.g. user123.

Step 2 — Server hands you a claim ticket
The server sends that ID back to your browser in a header:

#Set-Cookie: sessionid=user123; HttpOnly; Secure; SameSite=Strict

Think of this like a coat-check ticket. The server keeps your actual "coat" (your logged-in state) — you just walk away holding a numbered ticket.

Step 3 — Your browser carries the ticket automatically
From now on, every time you visit that same site, your browser automatically attaches:

#Cookie: sessionid=user123

to the request — you don't have to log in again on every page.

Step 4 — Server checks the ticket every time
On each request, the server sees user123, looks it up in its own storage, and goes "ah, this is usernam1, he's logged in" — then responds accordingly. The cookie itself has no real data in it — it's just a pointer. The actual "who you are" lives on the server, not in the cookie.

Why the flags on that Set-Cookie line matter:

>>HttpOnly — stops JavaScript on the page from reading the cookie. If a hacker sneaks in a malicious script (XSS), they still can't steal your session ID.
>>Secure — the cookie is only ever sent over HTTPS, never plain unencrypted HTTP, so it can't be sniffed on the network.
>>SameSite — controls whether the cookie gets sent when the request comes from a different website. This is what stops CSRF attacks, where another site tries to trick your browser into acting on your behalf.

How this connects to Burp Suite

Burp basically shows me everything I already learned in sections 2–3, just on a screen. Here's how I use it, step by step:

Step 1 — Proxy
Burp sits in the middle between my browser and the website. Every request I send and every response I get passes through here first — so I can literally see the raw request and response.

Step 2 — Repeater
I take one request I want to test, send it to Repeater, and manually change things — a header, a parameter, the method — then hit send and see what response comes back.

Step 3 — Intruder
Instead of testing one value at a time by hand, Intruder does it automatically — it takes my request, swaps in a list of different values one by one, and fires them all off for me.

Step 4 — Decoder
Sometimes data in a request looks like gibberish (Base64, URL-encoded, etc). Decoder converts it back into normal readable text.

Step 5 — HTTP History
This shows every request and response Burp has seen so far. I scroll through it looking for anything interesting — weird headers, cookies, or error codes.


Things I found confusing at first
Claude finished the response

Here's the simplest version:

7. How This Connects to Burp Suite

Burp basically shows me everything I already learned in sections 2–3, just on a screen. Here's how I use it, step by step:

Step 1 — Proxy: watch the traffic
Burp sits in the middle between my browser and the website. Every request I send and every response I get passes through here first — so I can literally see the raw request and response.

Step 2 — Repeater: edit and resend
I take one request I want to test, send it to Repeater, and manually change things — a header, a parameter, the method — then hit send and see what response comes back.

Step 3 — Intruder: automate it
Instead of testing one value at a time by hand, Intruder does it automatically — it takes my request, swaps in a list of different values one by one, and fires them all off for me.

Step 4 — Decoder: read scrambled data
Sometimes data in a request looks like gibberish (Base64, URL-encoded, etc). Decoder converts it back into normal readable text.

Step 5 — HTTP History: the full log
This shows every request and response Burp has seen so far. I scroll through it looking for anything interesting — weird headers, cookies, or error codes.

Want me to update the doc with this version?

Things I Found Confusing at First (and How I Resolved Them)
"Stateless but I stay logged in — contradiction?" No — the protocol itself has no memory, but the application layers state on top using cookies/tokens (section 6). Every request is still independently "stateless" from HTTP's point of view; it's just carrying a piece of state (the session ID) as data. Separating "the protocol" from "what the app builds on top of it" is what made this click.
401 vs 403 — I used to think they meant the same thing. 401 Unauthorized actually means "I don't know who you are" (missing/invalid authentication) — despite the confusing name, it's really about authentication. 403 Forbidden means "I know who you are, and you're not allowed" — that's authorization. Once I reframed it as authentication vs authorization, it stopped being confusing.
GET vs POST for "security" — I assumed POST was inherently more secure because params aren't in the URL. It isn't — POST just hides the data from the URL bar/browser history, it does nothing for encryption or access control. Anyone with the raw traffic (or Burp) sees the body just as easily. The only real protection is HTTPS + proper server-side auth checks.
Why editing Content-Length manually in Burp used to break my requests. If I hand-edited a body and left the old Content-Length, the server either truncated the body or hung waiting for more bytes. Now I let Burp auto-update it (it does, by default, in Repeater) instead of typing values myself.
OPTIONS felt pointless until CORS clicked. I originally skipped it as "not a real method." It became useful once I understood browsers automatically fire an OPTIONS preflight before certain cross-origin requests, and the Access-Control-Allow-* response headers are the server's answer to "are you allowed to do this from that origin." That's when OPTIONS + CORS headers started being something I check together instead of ignoring.
Cookies vs sessions vs tokens all blurred together. What fixed it: cookie = the transport mechanism (a header), session = the actual state, stored server-side, that the cookie value points to. A token (like a JWT) is a different pattern where the state itself is embedded in what's sent, self-contained and verifiable without a server-side lookup. Same goal (persist identity across stateless requests), different mechanism.make it look like i was writing it

Message collapsed

Here's a version that reads like actual personal notes rather than a polished write-up:

8. Things I Found Confusing at First (and How I Resolved Them)

"Wait, HTTP is stateless but I stay logged in... isn't that a contradiction?"
Yeah this one messed with me for a bit. Turns out no — the protocol really doesn't remember anything, that part's true. But the app built on top of it fakes memory by shoving a session ID into a cookie and making me send it back every time (section 6). So technically every request is still stateless on its own, it's just now carrying a little piece of state as data. Once I stopped mixing up "the protocol" and "what apps do on top of the protocol," this made way more sense.

401 vs 403 — I genuinely thought these were basically the same error for ages.
They're not. 401 Unauthorized is really the server saying "I have no idea who you are" — so it's actually about authentication, which is a weirdly misleading name for it honestly. 403 Forbidden means "I know exactly who you are, and no, you can't do that" — that's authorization. Once I started saying "authentication vs authorization" in my head instead of the status names, I stopped mixing them up.

I used to think POST was just... safer than GET.
Because the data isn't sitting in the URL, right? Nope. That was a dumb assumption on my part. POST just hides it from the address bar and browser history — it does nothing about encryption or who's allowed to see it. If someone's watching the raw traffic (or literally just opening Burp), they see the POST body just as easily as a GET param. The actual protection is HTTPS + the server checking permissions properly, not the method I picked.

Content-Length randomly breaking my requests in Burp — this annoyed me for a while.
I'd edit a body in Repeater, forget the Content-Length was still the old number, and the request would either get cut off or the server would just sit there waiting for more data. Eventually realized Burp updates this automatically for me by default — I don't need to touch it manually. Lesson: stop typing values I don't need to type.

OPTIONS felt like a completely useless method... until CORS clicked for me.
I used to just skip past it like it wasn't a "real" method. Then I learned browsers fire it automatically before certain cross-origin requests, as a sort of "hey am I even allowed to do this?" check — and the Access-Control-Allow-* headers coming back are literally the server's answer. Now when I see OPTIONS, I immediately go check the CORS headers next to it instead of ignoring it.

Cookies, sessions, tokens — these three words used to all blur into one concept for me.
What finally untangled it: a cookie is just the delivery method — it's a header, nothing fancier than that. A session is the actual data, sitting on the server, that the cookie value points back to. A token (like a JWT) is a totally different approach — the data itself is packed inside the token, so the server can verify it just by checking a signature, no lookup needed. Different mechanism, same end goal: proving who I am across requests that otherwise have no memory of me.

Want this swapped into the doc?

make it smaller

Here's a shorter version:

8. Things I Found Confusing at First (and How I Resolved Them)

Stateless but I stay logged in?
Thought this was a contradiction. It's not — HTTP itself still has no memory, the app just fakes it by passing a session ID back and forth in a cookie.

401 vs 403

Used to think these were the same. 401 = "I don't know who you are" (authentication). 403 = "I know who you are, but no" (authorization). Saying it that way finally made it stick.

Thought POST was "safer" than GET

It's not. POST just keeps data out of the URL bar — doesn't encrypt it or protect it. Anyone watching raw traffic sees it just fine. HTTPS + real auth checks are what actually matter.

Content-Length kept breaking my requests

Edited a body in Repeater, forgot to update Content-Length, request got cut off or hung. Turns out Burp auto-updates this for me — stopped touching it manually.

OPTIONS seemed pointless — until CORS clicked

Used to skip right past it. Then realized browsers fire it automatically before certain cross-origin requests, and the Access-Control-Allow-* headers are the server's answer. Now I check them together.

Cookies vs sessions vs tokens all blurred together

Cookie = the delivery method (a header). Session = the real data, stored server-side, that the cookie points to. Token (JWT) = data packed into the token itself, verified without a server lookup. Same goal, different mechanism.

References

>>>TryHackMe: HTTP in Detail
>>>PortSwigger Web Security Academy
