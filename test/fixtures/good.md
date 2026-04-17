You are an experienced technical writer specializing in developer documentation for mid-level backend engineers.

Your task: write a 500-word tutorial explaining how to implement JWT authentication in a Node.js Express API.

Audience and context:
- Readers know Node.js and Express but are new to JWT.
- They will copy-paste the examples into a working app.
- Assume Node 18+, Express 4.x, and the `jsonwebtoken` package.

Output format:
- Markdown with an H1 title and H2 headings for each step.
- Code blocks in JavaScript using ```js fences.
- Exactly one runnable example per section.

Cover these sections in this order:
1. Why JWT vs sessions (2-3 sentences)
2. Install dependencies (one `npm install` command)
3. Generate a signed token (10-20 lines of code)
4. Verify-token middleware (10-20 lines of code)
5. Protect a route (5-10 lines of code)
6. Three common pitfalls and how to avoid each

Constraints:
- Do NOT cover refresh tokens, OAuth, or Passport.js.
- Do NOT use any library other than `jsonwebtoken`.
- Do NOT include a full package.json or file-tree diagrams.
- Keep total length under 600 words.

End with a 3-item checklist of security best practices (e.g., "never hard-code the signing secret", "use HTTPS in production", "keep access-token lifetimes short").

Example of the tone we want:
"Here's how to generate a token: [code]. Notice we set `expiresIn` to `'1h'` — long enough for an active session but short enough to limit damage if the token leaks."
