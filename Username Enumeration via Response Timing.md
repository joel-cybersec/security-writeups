# Lab: [Username Enumeration via Response Timing]

**Platform:** PortSwigger Web Security Academy  
**Topic:** [Authentication]  
**Difficulty:** [Practitioner]  
**Status:** [Complete]

## Objective
Gain access to a user account by first enumerating the username from a wordlist followed by the password using HTTP response times.
## Vulnerability
- **What is it?** - The web application is vulnerable to username enumeration.
- **Why does it occur?** - Due to differing response times dependent upon if the username is valid or not, password hashing can occur or not occur. Thus, we can brute-force the username by observing server response times and if password hashing presumably occurs. In addition, flawed IP-based brute-force protection is in place by simple HTTP header modification.
- **What is the potential impact?** - This vulnerability allows an attacker to brute-force valid usernames which renders brute-forcing password attacks much easier which could then allow an attacker to gain access to unauthorised accounts. 
## Methodology
1. What did I inspect? - Intercepted the login request in Burp Suite and configured a sniper attack using Intruder and the provided user and password wordlists.
2. What was my hypothesis? - If the username in the payload is a valid one, the response time may be noticeably slower due to password hashing.
3. What did I change or test? - Changed the password to be long so that if my hypothesis is correct, password hashing should give a much longer response time than if it were invalid thus making the difference easier to observe.
4. What evidence confirmed the issue? - By inputting a 100 character password, response time of a single username caused more than a doubling of response-time when compared with other usernames on the word list thus highlighting this user for further investigation.

## Burp Suite Notes
- **Tool / attack type:** Intruder, pitchfork attack.
- **Relevant request or response:** Response received time.
- **Useful configuration:** Pitchfork attack. Keep other variables constant during the testing process. 100-character password to prevent noise during testing.
- **Mistakes and troubleshooting:** Initially used a normal password length which caused noise in the response times and thus made identifying the valid username difficult. After some research, increasing the character length to an absurd length made the valid username much more easy to spot. In addition, I had to investigate and research the IP-block to be able to circumvent it via adding an X-Forwarded-For header in the HTTP request which was incremented for each username attempted in the attack.

## Root Cause
What went wrong in the application's design or implementation?

Firstly, the IP-block was poorly configured. The web application trusted the client provided IP address in the X-Forwarded-For header in the HTTP request which allows an attacker to circumvent the protection completely by simply adding a header and incrementing the IP address. 

Secondly, the web application gave a different response times for valid and invalid usernames thus making brute-force attacks much easier. 
## Remediation
How should a developer fix this vulnerability?

To fix the IP-block, the developer should ensure that only the requesting IP is used for the connection rather than trusting HTTP headers. This would make circumventing the protection much more difficult or implement account-level rate limiting to prevent brute-forcing on a per account basis. Only trusted reverse proxies or load balancer's should be trusted to provide an X-Forwarded-For value. 

To remediate the web application brute-forcing vulnerability, a dummy hash that uses the same algorithm as regular passwords should be used in cases where the username is valid but the password is correct or incorrect. This would make spotting a correct password much more difficult and make it indistinguishable from an incorrect one via server response time and then return a generic failure message to prevent the attacker from determining a correct password. 
## Lessons Learned
- What did I learn? - How to use burp suite intruder and the different types of attacks it is capable of. I also learned about rate-limiting and the different types possible during my research. 
- How would I recognise this issue in a real assessment? - This issue could be recognised by first attempting to brute-force a username field and then checking for distinguishable response times using a valid dummy account. 
- What would I try differently next time? - I would immediately put the character length to something absurd when trying to isolate the issue. 

## References
- [PortSwigger lab link](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing)


## Review
- [x] Can I explain the vulnerability without looking at my notes?
- [x] Could I reproduce the technique from scratch?
- [x] Do I understand the remediation?