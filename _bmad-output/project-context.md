---
project_name: 'bookingSystem'
user_name: 'enci'
date: '2026-01-25'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'code_quality', 'workflow', 'critical_rules']
status: 'complete'
rule_count: 45
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

### Frontend
- **Vue.js** ^3.4.29 - MUST use Options API, NOT Composition API
- **Vue Router** ^4.3.3
- **Vite** ^5.3.1 - Build tool

### Backend
- **Express.js** ^4.19.2
- **MariaDB** ^3.3.1 (driver)
- **jsonwebtoken** ^9.0.2 - JWT Cookie authentication
- **Nodemailer** ^6.9.15 - Email notifications

### Testing
- **Vitest** ^1.6.0 - Frontend unit tests
- **Cypress** ^13.12.0 - E2E tests
- **Mocha** ^10.7.3 - Backend API tests

### Version Constraints
- Vue 3.4+ required (Options API style)
- No Vuex/Pinia - use Props/Emit only
- No Composition API imports (ref, reactive, computed from vue)

## Critical Implementation Rules

### Language-Specific Rules (JavaScript)

#### Import/Export Patterns
- Backend: CommonJS (`require`/`module.exports`)
- Frontend: ES Modules (`import`/`export`)
- Type: `"type": "module"` in frontend package.json

#### Async/Await Patterns
- Always use `async/await` over raw Promises
- API handlers: `async (req, res) => { try {...} catch {...} }`
- Database operations always wrapped in try-catch

#### Error Handling
- Backend API errors: Return `{ error: 'message' }` with appropriate HTTP status
- Never expose stack traces to client
- Log errors server-side before returning response

#### Variable Naming
- camelCase for all variables and functions
- UPPER_SNAKE_CASE for constants only
- Descriptive names: `seriesId` not `sid`

### Framework-Specific Rules

#### Vue.js Rules
- **MUST use Options API** - Never use Composition API (`setup()`, `ref()`, `reactive()`)
- **State Management**: Props down, Emit up - NO Vuex/Pinia
- **Component Structure**:
  ```javascript
  export default {
    name: 'ComponentName',
    props: { ... },
    emits: ['eventName'],
    data() { return { ... } },
    computed: { ... },
    methods: { ... }
  }
  ```
- **File naming**: camelCase.vue (e.g., `recurringForm.vue`)

#### Express.js Rules
- **API endpoint pattern**: `/api/{resource}` (kebab-case)
- **Route file naming**: camelCase.js (e.g., `recurringReservation.js`)
- **Response format**:
  - Success: Return data directly `{ reserve_id: 1, ... }`
  - Error: Return `{ error: 'message' }` with optional extra fields
- **Middleware order**: cors → cookie-parser → body-parser → routes

#### MariaDB Rules
- Use connection pool from `model/conn.js`
- Always use parameterized queries (prevent SQL injection)
- Batch operations MUST use transactions

### Testing Rules

#### Test Organization
- **Frontend unit tests**: `__test__/*.test.js` (Vitest)
- **Backend API tests**: `testing/*.js` (Mocha + Chai)
- **E2E tests**: `cypress/e2e/*.cy.js` (Cypress)

#### Test File Naming
- Frontend: `ComponentName.test.js` (PascalCase)
- Backend: `resourceName.js` (camelCase)
- E2E: `feature.cy.js` (camelCase)

#### Test Structure (Frontend - Vitest)
```javascript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'

describe('ComponentName', () => {
  it('should do something', () => {
    const wrapper = mount(Component)
    expect(wrapper.exists()).toBe(true)
  })
})
```

#### Test Structure (Backend - Mocha)
```javascript
const { expect } = require('chai')
const request = require('supertest')

describe('API Endpoint', function() {
  it('should return expected result', async function() {
    const res = await request(app).get('/api/resource')
    expect(res.status).to.equal(200)
  })
})
```

#### Coverage Requirements
- New features must include unit tests
- API endpoints must have integration tests
- Critical user flows must have E2E tests

### Code Quality & Style Rules

#### Database Naming Conventions
| Item | Convention | Example |
|------|------------|---------|
| Table names | PascalCase | `RecurringSeries` |
| Column names | snake_case | `series_id`, `occurrence_date` |
| Foreign keys | {table}_id | `series_id`, `room_id` |

#### API Naming Conventions
| Item | Convention | Example |
|------|------------|---------|
| Endpoints | kebab-case | `/api/recurring-reservation` |
| Query params | snake_case | `start_time`, `scope` |
| Path params | :id format | `/api/reservation/:id` |

#### Code Naming Conventions
| Item | Convention | Example |
|------|------------|---------|
| Backend Model | camelCase.js | `recurringSeries.js` |
| Backend API | camelCase.js | `recurringReservation.js` |
| Frontend Component | camelCase.vue | `recurringForm.vue` |
| Functions | camelCase | `createRecurringSeries` |
| Variables | camelCase | `seriesId`, `occurrenceDate` |

#### File Organization
- Backend models: `api/model/`
- Backend routes: `api/api/`
- Backend utilities: `api/utilities/`
- Frontend components: `MRBS-frontend/src/components/`
- Frontend assets: `MRBS-frontend/src/assets/`

### Development Workflow Rules

#### Git Workflow
- Main branch: `master`
- CI/CD: GitHub Actions (`.github/workflows/no_vpn_pipeline.yaml`)

#### Build Commands
- Frontend dev: `npm run dev` (Vite dev server)
- Frontend build: `npm run build`
- Frontend test: `npm run test:unit` (Vitest)
- Frontend E2E: `npm run test:e2e` (Cypress)
- Backend start: `npm run start`
- Backend test: `npm run test` (Mocha)

#### Docker
- Container orchestration: `docker-compose.yml`
- Build: `Dockerfile` in `src/`

#### Environment
- JWT secret via environment variable
- Database connection via environment variable
- CORS configuration for development/production

### Critical Don't-Miss Rules

#### Anti-Patterns to AVOID

| WRONG | CORRECT |
|-------|---------|
| `recurring_series` (table name) | `RecurringSeries` |
| `seriesId` (column name) | `series_id` |
| `/api/recurringSeries` | `/api/recurring-reservation` |
| `{ success: false, error: {} }` | `{ error: 'message' }` |
| Composition API (`ref`, `reactive`) | Options API (`data()`, `computed`) |
| Vuex/Pinia store | Props/Emit pattern |
| Multiple separate DB queries in loop | Single batch query with transaction |

#### Security Rules
- NEVER expose JWT secret in code
- ALWAYS use parameterized SQL queries
- NEVER return stack traces to client
- ALWAYS validate `privilege_level` for admin operations (1+ = admin)

#### Performance Gotchas
- Batch conflict check: ONE SQL query, not N queries
- Use connection pool, never create new connections per request
- Frontend: Don't re-fetch calendar data unnecessarily

#### Edge Cases to Handle
- Recurring reservation with 0 valid occurrences (all conflict)
- Edit scope `future` when it's the last occurrence
- Delete series when some occurrences already passed
- Timezone: Server uses system timezone, store dates as `DATE` not `DATETIME`

#### Date/Time Format
- API transmission: `YYYY-MM-DD HH:mm:ss`
- RRULE internal format handled by backend only

---

## Usage Guidelines

**For AI Agents:**
- Read this file before implementing any code
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Update this file if new patterns emerge

**For Humans:**
- Keep this file lean and focused on agent needs
- Update when technology stack changes
- Review quarterly for outdated rules
- Remove rules that become obvious over time

---

_Last Updated: 2026-01-25_
