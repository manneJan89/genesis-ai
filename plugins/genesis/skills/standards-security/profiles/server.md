# Security: server profile
APIs and backends that receive requests from clients you don't control: Node
(Express, NestJS, Fastify), Cloud Functions, Python, Go, and so on.

- **Schema-validate every request at the edge.** Body, query string, route
  params and headers each go through one schema before any handler logic: zod,
  NestJS `ValidationPipe` with `whitelist: true` and
  `forbidNonWhitelisted: true` on class-validator DTOs, Joi, pydantic. One
  pattern per project, applied to every route, not ad hoc `if` checks.
- **NoSQL operator injection (MongoDB, Mongoose).** A request value that reaches
  a query must be proven a plain string or number first. `{"password":
  {"$ne": ""}}` is valid JSON, and passed into `findOne` it matches every user:
  a login with no password. Rules:
  - validate the type at the edge (the schema above), so an object never
    arrives where a string is expected;
  - with Mongoose, also turn on `mongoose.set('sanitizeFilter', true)` as a
    second layer, never the only one;
  - never build a filter, sort or projection by passing a request object
    straight in (`find(req.query)`, `find(req.body.filter)`);
  - never let user input reach `$where`, `$function`, `$accumulator` or
    `mapReduce` JavaScript;
  - reject keys that start with `$` or contain `.` in anything stored.
- **SQL.** Parameterized queries or the ORM's safe paths only. Raw query
  helpers (`$queryRawUnsafe`, string-built `knex.raw`) never take user input.
- **Mass assignment.** Never `create(req.body)`, `update(id, req.body)` or
  `{...req.body}` into a document. Map the allowed fields explicitly or use a
  DTO whitelist.
- **Prototype pollution (JavaScript).** Never deep-merge request objects into
  existing objects (`lodash.merge`, `Object.assign` on nested input, hand-rolled
  merges); reject `__proto__`, `constructor` and `prototype` keys.
- **401 vs 403** used correctly, without revealing whether the resource exists
  to someone who may not see it.
- **Rate limiting** on login, signup, password reset, OTP and any expensive
  endpoint.
- **CORS:** an explicit origin list; never `*` together with credentials.
- **Outbound requests to user-supplied URLs (SSRF):** allowlist hosts; block
  internal and metadata addresses.
- **Files and paths:** uploads checked for type and size, stored outside the web
  root, never executed; user input never joins a filesystem path without
  normalizing it and checking it stays inside the intended folder.
- **Sessions and tokens:** they expire; cookies are `httpOnly`, `secure`,
  `sameSite`; refresh and logout invalidate on the server.

**Security tests for this profile** (on top of the universal ones): the operator
payload (`{"$ne": ""}`, `{"$gt": ""}`) against every endpoint that queries by a
request value, expecting a 400 and never a match; `__proto__` in a body;
rate-limit kicking in on auth endpoints.
