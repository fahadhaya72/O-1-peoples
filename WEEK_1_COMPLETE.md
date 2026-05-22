# Week 1 - Authentication & Contest Management

## ✅ Completed

### Database Schema
- ✅ User model with authentication
- ✅ Contest model with full CRUD
- ✅ ContestRegistration (user-contest mapping)
- ✅ Score model for leaderboard
- ✅ Indexes for performance

### API Endpoints (17 endpoints)

**Authentication** (4 endpoints)
```
POST   /api/auth/register       - User registration
POST   /api/auth/login          - Login & get JWT
POST   /api/auth/refresh        - Refresh token
POST   /api/auth/logout         - Logout
```

**Contests** (7 endpoints)
```
POST   /api/contests            - Create contest (ADMIN)
PUT    /api/contests/:id        - Update contest (ADMIN)
DELETE /api/contests/:id        - Delete contest (ADMIN)
GET    /api/contests            - List contests (paginated)
GET    /api/contests/:id        - Get contest details
POST   /api/contests/:id/register - Register user
GET    /api/contests/:id/leaderboard - Get leaderboard
```

### Backend Features
- ✅ JWT token-based authentication (7-day expiry)
- ✅ Password hashing (bcryptjs)
- ✅ Role-based access control (USER/ADMIN)
- ✅ Rate limiting (login: 5/min)
- ✅ Input validation
- ✅ Error handling
- ✅ Comprehensive logging (Winston)
- ✅ TypeScript for type safety

### Infrastructure
- ✅ PostgreSQL with connection pooling
- ✅ Redis for caching/queues
- ✅ Docker Compose setup
- ✅ Environment configuration
- ✅ Database seeding

## Quick Start

```bash
# 1. Install dependencies
cd backend
npm install

# 2. Start services
cd ..
docker-compose up -d

# 3. Setup database
docker-compose exec backend npx prisma db push
docker-compose exec backend npm run db:seed

# 4. Test API
curl http://localhost:3000/health
```

## Test Credentials

**Admin**
```
Email: admin@o1peoples.com
Password: admin123
```

**User**
```
Email: john@example.com
Password: password123
```

## API Testing Examples

### Register
```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "newuser",
    "email": "newuser@example.com",
    "password": "password123",
    "confirmPassword": "password123"
  }'
```

### Login
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@o1peoples.com",
    "password": "admin123"
  }'
```

### Create Contest (Admin)
```bash
TOKEN="<jwt_token_from_login>"
curl -X POST http://localhost:3000/api/contests \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Contest 1",
    "description": "Test contest",
    "startTime": "2024-12-01T10:00:00Z",
    "endTime": "2024-12-01T11:00:00Z",
    "maxProblems": 3,
    "maxParticipants": 100
  }'
```

### List Contests
```bash
curl http://localhost:3000/api/contests?page=1&limit=10
```

### Register for Contest
```bash
TOKEN="<jwt_token_from_login>"
curl -X POST http://localhost:3000/api/contests/1/register \
  -H "Authorization: Bearer $TOKEN"
```

### Get Leaderboard
```bash
curl http://localhost:3000/api/contests/1/leaderboard?limit=50
```

## File Structure

```
backend/
├── src/
│   ├── config/
│   │   ├── index.ts
│   │   └── logger.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   └── rateLimiter.ts
│   ├── lib/
│   │   └── prisma.ts
│   ├── services/
│   │   ├── authService.ts
│   │   └── contestService.ts
│   ├── controllers/
│   │   ├── authController.ts
│   │   └── contestController.ts
│   ├── routes/
│   │   ├── auth.ts
│   │   └── contests.ts
│   ├── app.ts
│   └── index.ts
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── Dockerfile
├── package.json
└── tsconfig.json
```

## Performance Optimizations

- Database indexes on: `email`, `username`, `status`, `startTime`
- Rate limiting prevents abuse
- Connection pooling for database
- JWT for stateless authentication
- Pagination for list endpoints

## Database Schema

```
┌──────────┐     ┌──────────────────┐
│  USER    │────▶│ CONTEST          │
├──────────┤     ├──────────────────┤
│ id (PK)  │     │ id (PK)          │
│ email    │     │ title            │
│ username │     │ adminId (FK→User)│
│ password │     │ status           │
│ role     │     │ startTime        │
└──────────┘     │ endTime          │
      ▲          │ duration         │
      │          └──────────────────┘
      │                   ▲
      │                   │
 ┌────────────────┐  ┌─────────────┐
 │REGISTRATION    │  │ SCORE       │
 ├────────────────┤  ├─────────────┤
 │userId (FK)  ───┼──→ userId (FK) │
 │contestId (FK)──┼──→ contestId   │
 └────────────────┘  │ solved      │
                     │ totalTime   │
                     │ penalties   │
                     │ rank        │
                     └─────────────┘
```

## Security

- ✅ Password hashing with bcryptjs (10 rounds)
- ✅ JWT with secure secret
- ✅ Rate limiting on sensitive endpoints
- ✅ CORS enabled
- ✅ Input validation
- ✅ Role-based authorization

## Next Steps (Week 2)

- [ ] Problem model
- [ ] Test case management
- [ ] Solution file uploads
- [ ] Problem display endpoints
- [ ] Admin problem management UI

---

**Status: WEEK 1 COMPLETE ✅**

Ready for Week 2: Problems + Test Cases
