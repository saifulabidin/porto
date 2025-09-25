Security Guideline & Best Practices Document
Version: 1.0
Date: September 26, 2025
Purpose: To establish a clear and actionable set of security standards and best practices for the entire lifecycle of the Cyberpunk Portfolio project. This document is mandatory reading for all development and deployment activities.

CHAPTER 1: Core Security Principles (The Zero Trust Mindset)
We operate under a "Zero Trust" model. No component of the system—frontend, backend, or database—is trusted by default. Every interaction must be authenticated and authorized.

Defense in Depth: Implement multiple layers of security controls. A failure in one layer should be caught by the next.

Principle of Least Privilege: Every user and service should only have the absolute minimum permissions required to perform its function. The admin user can manage content but cannot access the server shell via the application.

Secure by Default: Configure all services and frameworks with security as the priority. Disable unnecessary features and ports.

Never Trust User Input: All data originating from an external source (user browser, API call) must be treated as potentially malicious until validated and sanitized.

CHAPTER 2: Backend Security (Golang API)
2.1. Authentication & Authorization
Password Hashing: All admin passwords MUST be hashed using a strong, salted, and slow hashing algorithm like Bcrypt. Plain text passwords are strictly forbidden.

JWT for Sessions: Admin sessions are managed via JSON Web Tokens (JWT).

Payload: JWTs must contain the user_id and an exp (expiration) claim.

Algorithm: Use a strong algorithm like HS256 or RS256. The secret key MUST be a long, high-entropy string stored securely as an environment variable.

Expiration: Set a short-lived expiration for JWTs (e.g., 1-4 hours) and implement a refresh token mechanism if longer sessions are needed.

Middleware Protection: All admin API endpoints (/api/admin/*) MUST be protected by an authentication middleware that validates the JWT on every request.

2.2. Input Validation & Sanitization
Prevent SQL Injection: Exclusively use the ORM's (GORM) parameterized queries or prepared statements. Never construct SQL queries by concatenating strings with user input.

Prevent XSS: The backend API returns JSON, so the primary risk is stored XSS. The backend MUST validate all incoming data for expected types, lengths, and formats. Use a library like go-validator for struct validation.

Rate Limiting: Implement rate limiting on sensitive endpoints, especially /api/admin/login, to mitigate brute-force attacks.

2.3. API & Communication
HTTPS Only: The API MUST only be accessible over HTTPS. Configure the VPS web server (e.g., Nginx) to automatically redirect HTTP traffic to HTTPS.

CORS Configuration: Configure Cross-Origin Resource Sharing (CORS) on the backend to only allow requests from the specific Vercel frontend domain. Do not use a wildcard (*).

Error Handling: Never expose detailed stack traces or internal system information in error messages sent to the client. Return generic, logged error messages (e.g., "Internal Server Error").

CHAPTER 3: Frontend Security (Next.js)
3.1. Cross-Site Scripting (XSS) Prevention
Default Protection: React's JSX automatically escapes content rendered within elements, providing strong protection against XSS.

dangerouslySetInnerHTML: Use of this prop is STRICTLY FORBIDDEN unless absolutely necessary and the source HTML is sanitized using a library like DOMPurify.

3.2. Securely Handling Authentication Tokens
Storage: Store the JWT received from the backend in an httpOnly cookie. This makes the token inaccessible to client-side JavaScript, providing robust protection against XSS-based token theft. Do not store tokens in localStorage or sessionStorage.

CSRF Protection: When using cookies for authentication, Cross-Site Request Forgery (CSRF) becomes a risk. Implement CSRF protection by:

The backend sets a second "CSRF token" cookie that is not httpOnly.

The frontend reads this token and includes it in a custom request header (e.g., X-CSRF-Token) for all state-changing requests (POST, PUT, DELETE).

The backend verifies that the header value matches the cookie value.

3.3. Environment Variables
Client-Side Variables: Only variables prefixed with NEXT_PUBLIC_ are exposed to the browser. NEVER place secrets (API keys, etc.) in variables with this prefix.

Server-Side Variables: All secrets MUST be stored in server-side environment variables and accessed only in Server Components or API routes.

CHAPTER 4: Infrastructure & DevOps Security (VPS)
4.1. Server Hardening
Firewall: Enable and configure a firewall (e.g., ufw on Ubuntu). Only open necessary ports (e.g., 22 for SSH, 80/443 for HTTP/S, and the backend port).

SSH Security:

Disable password-based authentication. Use SSH key pairs only.

Disable root login (PermitRootLogin no).

Change the default SSH port from 22 to a non-standard port.

System Updates: Regularly update the server's operating system and all installed packages to patch known vulnerabilities.

4.2. Docker Security
Least Privilege: Run Docker containers as a non-root user.

Secret Management: Pass all secrets (database passwords, JWT secrets) to Docker containers via environment variables defined in a .env file, which is listed in .gitignore. Do not hardcode secrets in the Dockerfile or docker-compose.yml.

4.3. Dependency Management
Vulnerability Scanning: Regularly scan project dependencies for known vulnerabilities.

Frontend: npm audit or yarn audit.

Backend: govulncheck.

Update Policy: Keep dependencies reasonably up-to-date to ensure security patches are applied.
