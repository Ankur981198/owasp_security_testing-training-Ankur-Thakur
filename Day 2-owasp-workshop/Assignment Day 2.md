# Core Concept Questions

---

## 1. Testing Approach Change

### Question
In Juice Shop, the application guided testing through its built-in challenge tracker, making the target vulnerabilities more discoverable. In DVWA, no such guidance was provided, requiring a more tester-driven exploratory approach.

Describe how your testing approach changed between the two applications.

### Answer
OWASP Juice Shop is a great starting point for anyone beginning their security testing journey. The application provides continuous hints, pop-ups, and challenge tracking that clearly indicate when a vulnerability has been discovered. This guided approach makes learning much easier, especially for someone who is just getting familiar with common security vulnerabilities and how they can be exploited. It helps build confidence while introducing core concepts in a structured way.

DVWA, on the other hand, feels much closer to a real-world security testing experience. Once you have a foundational understanding of security testing concepts, DVWA becomes an excellent platform to practice independent exploration. Unlike Juice Shop, it does not guide you toward vulnerabilities, which means you need to think like a real tester, analyzing application behavior, identifying attack surfaces, crafting payloads, and validating findings on your own.

What makes DVWA particularly useful is the ability to change the application's security level. This allows you to revisit the same vulnerability with increasing complexity and experiment with different testing approaches, which helps deepen practical understanding. Overall, Juice Shop helps you learn the concepts, while DVWA helps you apply them in a way that feels much more realistic.

---

## 2. Manual vs Automated Testing

### Question
You performed this entire assessment manually without any automated scanner. What did manual testing allow you to find or understand that an automated tool would have missed or misrepresented?

### Answer
In my experience during testing, there were several scenarios where automated tools could have been useful for quickly identifying potential issues, but they would not have provided the same hands-on learning experience that manual testing offers. Manual testing helped me better understand how the application behaves, how vulnerabilities actually work, and how an attacker might think while exploring weaknesses.

For example, during the Blind SQL Injection assessment, extracting information through boolean-based responses was a slow and somewhat tedious process when done manually. However, that exercise was extremely valuable because it highlighted an important real-world lesson—not every vulnerability will directly expose the data you are looking for. In many cases, testers need to infer information step by step using observation, logic, and repeated trial-and-error techniques, such as identifying the database name one character at a time.

Similarly, in the Command Injection assessment, an automated tool might have detected the presence of command injection by testing a simple payload, but manual testing allowed for much deeper exploration. By chaining multiple commands together, I was able to understand the true extent of the vulnerability, including how much access could potentially be gained on the host system. This kind of practical exploration gives a much clearer picture of the real security impact, something automated tools alone may not fully demonstrate.

---

## 3. CEO-Level Critical Finding Explanation

### Question
Choose the single most critical finding from your assessment. Explain it as if you are presenting it to the CEO who has no technical background.

### Answer
The most critical finding identified during the assessment was the OS Command Injection vulnerability.

The most critical finding identified during the assessment was the SQL Injection vulnerability that allowed unauthorized database data extraction.

In my view, this had the highest impact on the overall security of the application because it allowed direct access to sensitive user information stored in the backend database. By executing carefully crafted SQL queries, an attacker could retrieve usernames, password hashes, and potentially other confidential application data without proper authorization. Since the passwords were protected using weak hashing methods rather than stronger modern encryption or hashing practices, the risk became even more severe, as attackers could potentially crack the credentials and gain unauthorized access to user accounts.

From a business perspective, this is a serious issue because user credentials and sensitive information should only ever be accessible to the intended users and authorized systems. Even for a non-technical audience, the impact is easy to understand, if an unauthorized individual can access customer login details or private data, it directly affects user trust, application security, and the organization’s reputation. In a real-world scenario, this type of vulnerability could lead to account compromise, data breaches, regulatory concerns, and significant business damage.

---

## 4. Defence in Depth

### Question
DVWA has a security level setting (Low, Medium, High, Impossible). You tested on Low. Based on what you found, what do you think Medium security does differently for the two most critical vulnerabilities you discovered?

### Answer
For the two most critical vulnerabilities identified—SQL Injection and OS Command Injection—the Medium security level would likely introduce stronger input validation and partial security controls.

For SQL Injection, Medium security would likely sanitize special SQL characters, restrict malformed inputs, or apply filtering mechanisms to reduce the effectiveness of direct injection payloads. However, unless secure parameterized queries are fully implemented, determined attackers may still find bypass techniques.

For OS Command Injection, Medium security would likely validate the allowed format of input values, block shell metacharacters such as `&&`, `;`, or `|`, and restrict execution of chained system commands. This would make exploitation more difficult but not necessarily impossible if validation is incomplete.

This demonstrates the importance of defence in depth, where multiple independent security controls reduce the likelihood of successful exploitation rather than relying on a single protection mechanism.

---