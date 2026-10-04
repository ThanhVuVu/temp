# Backend Development Learning Plan: "Prompt Laboratory"

**Project Concept:** A full-stack platform for AI developers to version, test, and optimize Claude prompts with cost tracking, A/B testing, and team collaboration.

**Learning Approach:** Complete from Zero, rapid execution (2-3 days per stage, ~4 weeks total)

**Target Student:** AI researcher/developer with no traditional backend experience

**Tech Stack (to be confirmed):**
- Backend: Node.js + TypeScript + Express (or FastAPI if Python preferred)
- Frontend: React (not focus, but needed for testing)
- Database: PostgreSQL
- LLM Integration: Claude API
- Queue: Bull/RabbitMQ (for async jobs)
- Vector DB: pgvector (for semantic search later)

---

## Stage 1: HTTP & Web Fundamentals (Days 1-2)

### Learning Objectives
- Understand how web requests work
- Grasp HTTP methods, status codes, headers, JSON
- Set up local dev environment
- Build first "Hello World" API endpoint

### Concepts to Master
- Request-Response cycle (what lives in request body, headers, query params)
- HTTP Methods: GET, POST, PUT, PATCH, DELETE semantics
- Status Codes: 200, 201, 400, 401, 403, 404, 500 — when to use each
- Headers: Content-Type, Authorization, CORS
- JSON format and serialization
- Tools: curl, Postman, or similar

### Hands-On Exercise
1. **Manual HTTP Request (using curl)**
   - Send raw HTTP requests to public API (example: JSONPlaceholder)
   - Observe request headers, response headers, body format
   - Experiment with different methods (GET vs POST)

2. **First Express Server**
   - Create minimal Express app with TypeScript
   - Add single endpoint: `GET /health` returns `{ status: "ok" }`
   - Test with curl/Postman — see raw HTTP flow
   - Add console logging to see request object structure

3. **Build Simple "Echo" API**
   - `POST /echo` accepts JSON body `{ message: "hello" }`, returns `{ received: "hello", timestamp: ... }`
   - Practice reading request body, status codes, JSON response
   - No database yet — just in-memory

### Why This Matters in Enterprise
- CORS errors, authentication headers, status code confusion — you'll see these daily
- Learning HTTP first means you'll debug API issues way faster
- Foundation for understanding microservices communication

### Checkpoint
- [ ] Can explain what's in an HTTP request (method, headers, body)
- [ ] Can write curl command to POST JSON and interpret response
- [ ] Have working Express server with 2-3 endpoints
- [ ] Understand why `POST` for create, `GET` for fetch, `PATCH` for update

---

## Stage 2: Choose Stack & Setup Project Structure (Days 3-4)

### Learning Objectives
- Set up TypeScript + Node.js properly
- Understand why project structure matters
- Create scalable folder layout from day one
- Set up Git + basic CI/CD

### Concepts to Master
- TypeScript: interfaces, types, tsconfig setup
- Module organization: routes, controllers, services, models
- Environment variables and config management
- Git workflows (main branch, feature branches)
- Basic npm/yarn scripts

### Hands-On Exercise
1. **Scaffold Project with Proper Structure**
   ```
   prompt-lab/
   ├── src/
   │   ├── controllers/     (handle HTTP requests)
   │   ├── services/        (business logic)
   │   ├── routes/          (define endpoints)
   │   ├── middleware/       (auth, logging, etc)
   │   ├── config/          (env variables)
   │   ├── types/           (TypeScript interfaces)
   │   └── index.ts         (entry point)
   ├── tests/
   ├── .env.example
   ├── tsconfig.json
   └── package.json
   ```

2. **Refactor "Echo API" into this structure**
   - Move route into `routes/echo.ts`
   - Create `controllers/echoController.ts` (request handling logic)
   - Create `services/echoService.ts` (business logic)
   - See how separation makes code cleaner

3. **Environment Setup**
   - Create `.env.example` with all vars needed
   - Load from `.env` using `dotenv`
   - Understand why secrets never go in code

### Why This Matters in Enterprise
- Enterprise codebases are 50,000+ lines — if not structured, impossible to maintain
- Team onboarding depends on clear structure
- Microservices communication often breaks because of unclear responsibility layers

### Checkpoint
- [ ] Project compiles with TypeScript, no errors
- [ ] Can add a new endpoint by following existing pattern (no copy-paste confusion)
- [ ] `.env` properly configured, secrets not in git

---

## Stage 3: REST APIs & Validation (Days 5-7)

### Learning Objectives
- Build CRUD endpoints properly
- Validate all input
- Return correct status codes
- Handle errors consistently

### Concepts to Master
- REST conventions (resource-based URLs)
- Input validation (required fields, types, constraints)
- Error response format
- Request body schema validation
- Idempotency (same request = same result)

### Hands-On Exercise: Build "Prompt" CRUD endpoints

This is the first real feature of Prompt Laboratory.

**Endpoints to create:**
```
GET    /api/prompts           → list all prompts (paginated)
POST   /api/prompts           → create new prompt
GET    /api/prompts/:id       → fetch one prompt
PATCH  /api/prompts/:id       → update prompt
DELETE /api/prompts/:id       → delete prompt
```

**Prompt object schema:**
```typescript
interface Prompt {
  id: string;
  title: string;
  content: string;           // the actual prompt text
  version: number;           // starts at 1
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;         // user ID (hardcoded for now)
  tags: string[];
  isPublic: boolean;
}
```

**Validation rules:**
- `title`: required, 3-100 characters
- `content`: required, 10-10000 characters
- `tags`: optional, max 5 tags
- `isPublic`: optional, defaults false

**Implementation steps:**
1. Create `services/promptService.ts` with in-memory storage (no DB yet)
   ```typescript
   class PromptService {
     private prompts: Map<string, Prompt> = new Map();
     
     create(data: CreatePromptDTO): Prompt { ... }
     getById(id: string): Prompt | null { ... }
     list(page: number): Prompt[] { ... }
     update(id: string, data: UpdatePromptDTO): Prompt { ... }
     delete(id: string): void { ... }
   }
   ```

2. Create `controllers/promptController.ts` that:
   - Calls service methods
   - Returns correct status codes (201 for create, 200 for get/update, 204 for delete)
   - Catches errors and returns 400/404/500

3. Create `routes/prompts.ts` to define all endpoints

4. Add **validation middleware** using `joi` or `zod`:
   ```typescript
   router.post('/api/prompts',
     validateBody(createPromptSchema),
     promptController.create
   );
   ```

5. Test all endpoints with Postman:
   - Create prompt → should return 201 with `id`
   - Get it back → should return 200
   - Update → should return 200
   - Delete → should return 204
   - Try invalid data → should return 400 with error message

### Why This Matters in Enterprise
- 80% of bugs come from validating wrong (or not at all)
- When 100 engineers hit your API, half will send invalid data intentionally or by mistake
- Idempotency prevents accidental double-charges, duplicate records

### Checkpoint
- [ ] All 5 CRUD endpoints working
- [ ] Invalid input returns 400 with clear error message
- [ ] Create returns 201, update returns 200, delete returns 204
- [ ] Can handle pagination (offset/limit query params)

---

## Stage 4: Databases & Data Modeling (Days 8-10)

### Learning Objectives
- Design relational schema
- Write SQL queries
- Understand transactions and constraints
- Set up PostgreSQL locally

### Concepts to Master
- Tables, rows, columns, primary keys, foreign keys
- Relationships: 1-to-many, many-to-many
- Indexes for query performance
- Transactions and ACID guarantees
- Migrations (version-controlling schema changes)

### Hands-On Exercise: Implement Prompt + User + Experiment tables

**Data Model for Prompt Laboratory:**

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(50) DEFAULT 'user',  -- 'admin', 'user', 'viewer'
  createdAt TIMESTAMP DEFAULT NOW()
);

-- Prompts table (what we built in Stage 3, now in DB)
CREATE TABLE prompts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  content TEXT NOT NULL,
  version INT DEFAULT 1,
  createdBy UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  isPublic BOOLEAN DEFAULT FALSE,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);

-- PromptVersions table (audit trail)
CREATE TABLE prompt_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  promptId UUID NOT NULL REFERENCES prompts(id) ON DELETE CASCADE,
  version INT NOT NULL,
  content TEXT NOT NULL,
  changedBy UUID NOT NULL REFERENCES users(id),
  changeReason VARCHAR(500),
  createdAt TIMESTAMP DEFAULT NOW()
);

-- Experiments table (A/B tests)
CREATE TABLE experiments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  promptAId UUID NOT NULL REFERENCES prompts(id),
  promptBId UUID NOT NULL REFERENCES prompts(id),
  status VARCHAR(50) DEFAULT 'running',  -- 'running', 'completed'
  createdBy UUID NOT NULL REFERENCES users(id),
  createdAt TIMESTAMP DEFAULT NOW()
);

-- ExperimentResults table (each run stores cost, latency, output)
CREATE TABLE experiment_results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  experimentId UUID NOT NULL REFERENCES experiments(id) ON DELETE CASCADE,
  promptVersion CHAR(1) NOT NULL,  -- 'A' or 'B'
  input TEXT NOT NULL,
  output TEXT NOT NULL,
  costUSD DECIMAL(10, 6),
  latencyMs INT,
  createdAt TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_prompts_createdBy ON prompts(createdBy);
CREATE INDEX idx_experiments_createdBy ON experiments(createdBy);
CREATE INDEX idx_experiment_results_experimentId ON experiment_results(experimentId);
```

**Implementation steps:**

1. **Set up database locally**
   - Install PostgreSQL, create database `prompt_lab_dev`
   - Use `psql` to run schema SQL above

2. **Install ORM: Prisma or TypeORM**
   - For simplicity, use Prisma
   - Create `prisma/schema.prisma` mirroring SQL above
   - Run `prisma migrate dev --name init`

3. **Replace in-memory PromptService with DB queries**
   ```typescript
   class PromptService {
     async create(data: CreatePromptDTO): Promise<Prompt> {
       return prisma.prompt.create({
         data: {
           title: data.title,
           content: data.content,
           createdBy: data.userId,  // hardcoded for now
         }
       });
     }
     
     async getById(id: string): Promise<Prompt> {
       return prisma.prompt.findUnique({ where: { id } });
     }
     
     async list(page: number): Promise<Prompt[]> {
       return prisma.prompt.findMany({
         skip: (page - 1) * 10,
         take: 10,
       });
     }
   }
   ```

4. **Test endpoints again** — should work exactly like Stage 3, but now data persists in DB

### Why This Matters in Enterprise
- Data relationships define reliability — bad schema = cascade failures
- Indexes make difference between 10ms query and 10s query
- Migrations track schema history — critical for team collaboration

### Checkpoint
- [ ] PostgreSQL running with all tables created
- [ ] CRUD endpoints use database (not in-memory)
- [ ] Can query by ID, get 404 if not found
- [ ] Version table tracks every change (audit trail started)

---

## Stage 5: Authentication & Authorization (Days 11-13)

### Learning Objectives
- Implement user signup/login
- Understand sessions vs tokens
- Protect endpoints by user role
- Learn audit logging (who did what, when)

### Concepts to Master
- Password hashing (bcrypt, not plain text)
- JWT tokens (when to use, how to store)
- Sessions with cookies
- Ownership checks (user can only modify own prompts)
- Role-based access (admin vs user vs viewer)
- Audit logging (every important action is logged)

### Hands-On Exercise: Add User System + Auth

**New endpoints:**
```
POST   /api/auth/signup         → register new user
POST   /api/auth/login          → login, get JWT token
POST   /api/auth/logout         → invalidate token
GET    /api/auth/me             → current user info
PATCH  /api/users/:id           → update user (admin only)
```

**Implementation:**

1. **Signup/Login flow**
   ```typescript
   // routes/auth.ts
   router.post('/signup', async (req, res) => {
     const { email, password, name } = req.body;
     
     // Validate
     if (!email || !password) return res.status(400).json(...);
     
     // Hash password
     const hashedPassword = await bcrypt.hash(password, 10);
     
     // Create user
     const user = await prisma.user.create({
       data: { email, name, password: hashedPassword }
     });
     
     // Generate JWT
     const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);
     
     return res.status(201).json({ token, user });
   });
   ```

2. **Auth middleware**
   ```typescript
   // middleware/auth.ts
   export const requireAuth = (req, res, next) => {
     const token = req.headers.authorization?.split(' ')[1];
     if (!token) return res.status(401).json({ error: 'No token' });
     
     try {
       const decoded = jwt.verify(token, process.env.JWT_SECRET);
       req.userId = decoded.userId;
       next();
     } catch {
       return res.status(401).json({ error: 'Invalid token' });
     }
   };
   ```

3. **Ownership check**
   ```typescript
   // In promptController.update
   const prompt = await promptService.getById(id);
   if (prompt.createdBy !== req.userId) {
     return res.status(403).json({ error: 'Not owner' });
   }
   ```

4. **Audit logging**
   ```typescript
   // Create audit log entry for every important action
   await prisma.auditLog.create({
     data: {
       userId: req.userId,
       action: 'PROMPT_UPDATED',
       resourceId: prompt.id,
       timestamp: new Date(),
       details: { oldVersion, newVersion }
     }
   });
   ```

5. **Test flow:**
   - Signup → get token
   - Create prompt with token → should work
   - Try create without token → 401
   - Try update someone else's prompt → 403
   - Try admin action as user → 403

### Why This Matters in Enterprise
- Authentication is the gate — weak auth means data breaches
- Audit logging is **mandatory** for compliance (GDPR, SOC 2, etc.)
- Most enterprise bugs happen at authorization boundary (user seeing data they shouldn't)

### Checkpoint
- [ ] Can signup and get JWT token
- [ ] Endpoints require auth (401 without token)
- [ ] Can only modify own prompts (403 if not owner)
- [ ] Audit log records who did what (view in DB)

---

## Stage 6: Async Jobs & Queues (Days 14-16)

### Learning Objectives
- Understand why async matters (don't block requests on slow work)
- Implement job queue for Claude API calls
- Handle failures and retries
- Monitor job status

### Concepts to Master
- Request-response cycle must stay fast (<200ms)
- Long operations (API calls, file processing) go in background
- Job queues store work, workers process it
- Retries for reliability
- Job status tracking (pending, processing, done, failed)

### Hands-On Exercise: Add Claude API Integration with Async Processing

**Concept: When user requests Claude to classify/analyze a prompt, instead of blocking, queue the job.**

**New endpoints:**
```
POST   /api/prompts/:id/analyze       → queue analysis job, return immediately
GET    /api/jobs/:jobId               → check job status
```

**Implementation:**

1. **Set up Bull queue (Redis-based job queue)**
   ```typescript
   import Queue from 'bull';
   
   export const analysisQueue = new Queue('prompt-analysis', {
     redis: { host: 'localhost', port: 6379 }
   });
   ```

2. **Queue a job when user requests analysis**
   ```typescript
   // controllers/promptController.ts
   async analyzePrompt(req, res) {
     const { id } = req.params;
     
     // Create job in queue (returns immediately)
     const job = await analysisQueue.add(
       { promptId: id, userId: req.userId },
       { attempts: 3, backoff: { type: 'exponential', delay: 2000 } }
     );
     
     // Store job reference in DB
     await prisma.job.create({
       data: {
         id: job.id.toString(),
         status: 'pending',
         promptId: id,
         type: 'ANALYSIS'
       }
     });
     
     return res.status(202).json({ jobId: job.id, status: 'queued' });
   }
   ```

3. **Worker processes job in background**
   ```typescript
   // workers/analysisWorker.ts
   analysisQueue.process(async (job) => {
     const { promptId, userId } = job.data;
     
     // Fetch prompt
     const prompt = await prisma.prompt.findUnique({ where: { id: promptId } });
     
     // Call Claude API to analyze the prompt
     const analysis = await callClaudeAPI({
       model: 'claude-opus-4-20250805',
       messages: [{
         role: 'user',
         content: `Analyze this prompt and provide: 
           1. Clarity (1-10)
           2. Specificity (1-10)  
           3. Suggestions for improvement
           \n\nPrompt: ${prompt.content}`
       }]
     });
     
     // Store result
     await prisma.promptAnalysis.create({
       data: {
         promptId,
         analysis: analysis.content,
         costUSD: analysis.usage.cost,
         createdAt: new Date()
       }
     });
     
     // Update job status
     await prisma.job.update({
       where: { id: job.id.toString() },
       data: { status: 'completed' }
     });
   });
   ```

4. **Check job status**
   ```typescript
   // controllers/jobController.ts
   async getJobStatus(req, res) {
     const { jobId } = req.params;
     const job = await prisma.job.findUnique({ where: { id: jobId } });
     
     return res.json({
       id: job.id,
       status: job.status,
       result: job.result  // filled when complete
     });
   }
   ```

5. **Test flow:**
   - POST /api/prompts/123/analyze → get 202 + jobId immediately
   - GET /api/jobs/{jobId} → initially shows "pending"
   - Wait 5 seconds
   - GET /api/jobs/{jobId} → now shows "completed" + analysis result

### Why This Matters in Enterprise
- Claude API can take 5-30 seconds — if you block request, user sees hanging
- Queues decouple request handling from processing (scale independently)
- Retries handle transient failures (Claude API overload, network hiccup)
- Real enterprise systems do 50+ async operations daily

### Checkpoint
- [ ] Can queue job without blocking request
- [ ] Can check job status
- [ ] Job completes and stores Claude API result
- [ ] Failed jobs retry automatically

---

## Stage 7: Observability & Logging (Days 17-18)

### Learning Objectives
- Add structured logging
- Monitor API performance
- Catch errors early
- Understand what's happening in production

### Concepts to Master
- Structured logging (JSON, not text)
- Logging levels (debug, info, warn, error)
- Request tracing (track request ID through all logs)
- Performance monitoring (response time, error rate)
- Error tracking

### Hands-On Exercise: Add Winston Logger + Basic Monitoring

**Implementation:**

1. **Structured logging**
   ```typescript
   // config/logger.ts
   import winston from 'winston';
   
   export const logger = winston.createLogger({
     format: winston.format.json(),
     transports: [
       new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
       new winston.transports.File({ filename: 'logs/combined.log' }),
       new winston.transports.Console()  // dev only
     ]
   });
   ```

2. **Request logging middleware**
   ```typescript
   // middleware/logging.ts
   export const logRequest = (req, res, next) => {
     const requestId = randomUUID();
     req.id = requestId;
     
     const startTime = Date.now();
     
     res.on('finish', () => {
       const duration = Date.now() - startTime;
       logger.info({
         requestId,
         method: req.method,
         path: req.path,
         status: res.statusCode,
         durationMs: duration,
         userId: req.userId || 'anonymous'
       });
     });
     
     next();
   };
   ```

3. **Error logging**
   ```typescript
   // middleware/errorHandler.ts
   export const errorHandler = (err, req, res, next) => {
     logger.error({
       requestId: req.id,
       error: err.message,
       stack: err.stack,
       userId: req.userId,
       path: req.path
     });
     
     res.status(500).json({ error: 'Internal server error' });
   };
   ```

4. **View logs**
   - Check `logs/combined.log` for all events
   - Filter by `requestId` to trace a user's request through system
   - Monitor error count (setup alert if > 5 errors/min)

### Why This Matters in Enterprise
- You'll deploy and something breaks silently — good logging saves hours of debugging
- Compliance requires audit trail — logs are evidence
- Performance issues hidden until logging shows which endpoint is slow

### Checkpoint
- [ ] All requests logged with requestId
- [ ] Errors captured with stack trace
- [ ] Can find which user triggered which action (audit trail)

---

## Stage 8: Testing (Days 19-20)

### Learning Objectives
- Write tests that protect against regressions
- Test API behavior, not just functions
- Understand test coverage

### Concepts to Master
- Unit tests (one function)
- Integration tests (full API flow)
- Test setup/teardown
- Mocking external services (Claude API)
- Test database (isolated from production)

### Hands-On Exercise: Write Integration Tests

**Example tests:**

```typescript
// tests/prompts.test.ts
import request from 'supertest';
import app from '../src/app';

describe('Prompt API', () => {
  let userId: string;
  let token: string;
  
  beforeEach(async () => {
    // Create test user
    const signupRes = await request(app)
      .post('/api/auth/signup')
      .send({ email: 'test@example.com', password: 'pass123', name: 'Test' });
    
    token = signupRes.body.token;
    userId = signupRes.body.user.id;
  });
  
  it('should create a prompt', async () => {
    const res = await request(app)
      .post('/api/prompts')
      .set('Authorization', `Bearer ${token}`)
      .send({
        title: 'Test Prompt',
        content: 'Classify this document: ...'
      });
    
    expect(res.status).toBe(201);
    expect(res.body.id).toBeDefined();
    expect(res.body.createdBy).toBe(userId);
  });
  
  it('should not allow unauthenticated create', async () => {
    const res = await request(app)
      .post('/api/prompts')
      .send({ title: 'Test', content: 'Test content' });
    
    expect(res.status).toBe(401);
  });
  
  it('should not allow user to modify others prompts', async () => {
    // Create prompt as user 1
    const prompt1Res = await request(app)
      .post('/api/prompts')
      .set('Authorization', `Bearer ${token}`)
      .send({ title: 'Prompt 1', content: 'Content 1' });
    
    // Try to update as user 2
    const signup2 = await request(app)
      .post('/api/auth/signup')
      .send({ email: 'test2@example.com', password: 'pass123', name: 'Test 2' });
    
    const res = await request(app)
      .patch(`/api/prompts/${prompt1Res.body.id}`)
      .set('Authorization', `Bearer ${signup2.body.token}`)
      .send({ title: 'Hacked' });
    
    expect(res.status).toBe(403);
  });
});
```

**Run tests:**
```bash
npm test
```

### Why This Matters in Enterprise
- Regressions (breaking existing features) are the #1 cause of production incidents
- Tests let you refactor with confidence
- Team of 10 engineers = must have tests to prevent stepping on each other

### Checkpoint
- [ ] 10+ tests covering critical paths
- [ ] Can run tests in isolation (test DB)
- [ ] CI pipeline runs tests automatically

---

## Stage 9: Deployment & Environment Management (Days 21-22)

### Learning Objectives
- Deploy backend to production
- Manage environment variables
- Set up monitoring
- Handle zero-downtime updates

### Concepts to Master
- Deployment platforms (Fly.io, Railway, Heroku, AWS)
- Environment-specific configs (dev, staging, prod)
- Database migrations in production
- Container basics (Docker)

### Hands-On Exercise: Deploy to Fly.io

**Steps:**

1. **Create Dockerfile**
   ```dockerfile
   FROM node:20-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci --only=production
   COPY dist ./dist
   EXPOSE 3000
   CMD ["node", "dist/index.js"]
   ```

2. **Create fly.toml**
   ```toml
   app = "prompt-lab"
   
   [[services]]
   internal_port = 3000
   protocol = "tcp"
   
   [env]
   DATABASE_URL = "postgresql://..."
   JWT_SECRET = "xxxxx"
   CLAUDE_API_KEY = "sk-xxxxx"
   ```

3. **Deploy**
   ```bash
   npm run build
   fly deploy
   ```

4. **Test**
   - POST to https://prompt-lab.fly.dev/api/auth/signup
   - Should work exactly like localhost
   - Check logs: `fly logs`

### Why This Matters in Enterprise
- Code on your laptop ≠ code in production (environment, scale, load)
- First production deployment teaches more than 10 local development sessions
- Many "bugs" only appear in production (timezone issues, concurrent load, etc.)

### Checkpoint
- [ ] Backend running on production URL
- [ ] Can create prompts via deployed API
- [ ] Database accessible from production
- [ ] Logs visible in fly.io dashboard

---

## Stage 10: Advanced Patterns (Days 23-24+)

### Learning Objectives (Pick 2-3 to go deep)

#### A. A/B Testing & Experiments
- Queue experiments (comparing prompt A vs B)
- Collect results (Claude output, cost, latency)
- Statistical analysis (which prompt performs better?)
- Dashboard showing experiment results

#### B. Caching & Performance
- Cache frequently accessed prompts
- Reduce database queries
- Monitor cache hit rate

#### C. Semantic Search with Embeddings
- Generate embeddings for each prompt (using Claude API)
- Store in pgvector
- Search by meaning, not just keywords

#### D. Cost Tracking & Optimization
- Track Claude API spending
- Per-user cost allocation
- Alerts if spending > budget
- Cost optimization suggestions

### Implementation Hints for Cost Tracking

```typescript
// services/costTracker.ts
class CostTracker {
  async logCost(data: {
    userId: string;
    operation: 'ANALYSIS' | 'EXPERIMENT';
    costUSD: number;
    tokens: { input: number; output: number };
  }) {
    await prisma.costLog.create({ data });
    
    // Check if user exceeded budget
    const monthlySpend = await getTotalSpendThisMonth(data.userId);
    if (monthlySpend > USER_BUDGET) {
      notifyUser(data.userId, 'Budget exceeded');
    }
  }
}
```

---

## Project Milestones

| Milestone | Stages | Output |
|-----------|--------|--------|
| **Week 1: Foundations** | 1-3 | Working REST API for prompts (CRUD) |
| **Week 2: Core Features** | 4-5 | User auth + prompts stored in DB + audit logging |
| **Week 3: Async & Scaling** | 6-7 | Claude API integration + job queue + structured logging |
| **Week 4: Production-Ready** | 8-10 | Tests + deployed + cost tracking + semantic search |

---

## Key Patterns to Recognize (Enterprise Lessons)

As you build, notice these patterns:

1. **Request → Validate → Process → Respond** (every endpoint follows this)
2. **Separation of concerns** (routes vs controllers vs services vs database)
3. **Async operations must be queued** (don't block on slow external services)
4. **Authentication != Authorization** (who you are vs what you can do)
5. **Audit logging is non-negotiable** (who did what, when, why)
6. **Tests catch the obvious bugs, logging catches the mysterious ones**
7. **First deployment is harder than the 100th**

---

## Handoff to Opus 5.5

When you pass this to Opus 5.5, ask it to:

1. **Expand each stage** with more code examples (real, runnable examples)
2. **Create GitHub repo structure** with skeleton files and comments
3. **Write setup instructions** (how to install deps, run locally, etc.)
4. **Create detailed exercise descriptions** for each stage
5. **Generate sample test files** showing what tests to write
6. **Create a progress checklist** (track completion per stage)

---

## Success Criteria

After finishing all 10 stages, you should be able to:

- [ ] Explain every HTTP concept to a junior developer
- [ ] Design a database schema from requirements
- [ ] Build a full-stack API with auth, validation, async jobs
- [ ] Deploy to production and debug when things break
- [ ] Write tests that actually prevent bugs
- [ ] Monitor a service in production (logs, errors, performance)
- [ ] Estimate effort for new backend features
- [ ] Mentor someone else through this same journey

