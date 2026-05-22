# O-1 Peoples - Competitive Programming Platform

A scalable competitive programming platform supporting 400+ concurrent users with real-time judging.

## Architecture Overview

```
INTERNET
    │
    ▼
┌────────────────┐
│     NGINX      │ (Load Balancer)
└────────┬───────┘
         │
    ┌────┴────┐
    ▼         ▼
 Backend    Backend
(Express)  (Express)
    │         │
    ���────┬────┘
         ▼
    ┌──────────┐
    │PostgreSQL│
    └────┬─────┘
         │
    ┌────┴────┐
    ▼         ▼
  Redis    Judge Workers
 (Queue)  (Docker)
```

## Tech Stack

- **Frontend**: React (Vite), Tailwind, Monaco Editor, Socket.IO
- **Backend**: Node.js, Express, Prisma, PostgreSQL
- **Queue**: Redis + BullMQ
- **Judge**: Docker containers (Python 3.13, OpenJDK 21, GCC 14, G++ 14)
- **Deployment**: Docker + Kubernetes

## Capacity

- Users Registered: 5,000+
- Concurrent Contest: 400
- Peak Submit Burst: 100–150 submits/min
- Target Uptime: 99%

## Implementation Timeline

- **Week 1**: Auth + Contest Management
- **Week 2**: Problems + Test Cases
- **Week 3**: Judge System
- **Week 4**: Queue & Job Processing
- **Week 5**: Leaderboard
- **Week 6**: Load Testing
- **Week 7**: Production Deployment

## Directory Structure

```
O-1-peoples/
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── middleware/
│   │   ├── config/
│   │   └── app.ts
│   ├── prisma/
│   │   └── schema.prisma
│   ├── docker/
│   └── package.json
├── frontend/
│   ├── src/
│   └── package.json
├── judge/
│   ├── executor.js
│   ├── docker/
│   └── package.json
├── storage/
│   └── problems/
│       └── problem_1/
├── docker-compose.yml
└── .env.example
```

## Getting Started

1. Clone repository
2. Create `.env` from `.env.example`
3. Install dependencies
4. Run `docker-compose up`

## Database Schema

See `SCHEMA.md` for complete database design

## API Documentation

See `API.md` for endpoint documentation

## Deployment

See `DEPLOYMENT.md` for production setup
