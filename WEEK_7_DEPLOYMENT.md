# Week 7 - Production Deployment

## Deployment Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   INTERNET / DNS                         │
└────────────────────────┬─────────────────────────────────┘
                         │
                    https://o1peoples.com
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────▼────┐     ┌────▼────┐    ┌────▼────┐
    │ CDN     │     │ DNS     │    │ Firewall│
    │CloudFront
    │         │     │Route53  │    │ WAF     │
    └────┬────┘     └────┬────┘    └────┬────┘
         │               │              │
         └───────────────┼──────────────┘
                         │
                    ┌────▼─────┐
                    │ Load     │
                    │ Balancer │
                    │ (NGINX)  │
                    └────┬─────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────▼────┐     ┌────▼────┐    ┌────▼────┐
    │Backend #1    │Backend #2    │Backend #3
    │(4 vCPU)      │(4 vCPU)      │(4 vCPU)
    └────┬────┘     └────┬────┘    └────┬────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
            ┌────────────┼────────────┐
            │            │            │
       ┌────▼────┐ ┌────▼────┐ ┌────▼────┐
       │Database │ │ Redis   │ │ Judge  │
       │Master   │ │ Cache   │ │Workers │
       │         │ │         │ │(8)    │
       └────┬────┘ └─────────┘ └────────┘
            │
       ┌────▼────┐
       │DB Replica
       │ (Read)  │
       └─────────┘
```

## Hardware Specifications

### Production Server
```
CPU: 8 vCPU (2.5+ GHz)
RAM: 16 GB
Disk: 500 GB SSD (RAID 1)
Network: 10 Gbps

Allocation:
- 2 vCPU → Backend
- 2 vCPU → Database
- 4 vCPU → Judge Workers
```

### Database Server (Separate)
```
CPU: 4 vCPU
RAM: 8 GB
Disk: 1 TB SSD (RAID 1)
Network: 10 Gbps
```

### Judge Worker Pool (3-4 Machines)
```
CPU: 4 vCPU each
RAM: 8 GB each
Disk: 200 GB SSD each
Docker containers: 4-6 per machine
```

## Deployment Platforms

### Option 1: AWS
```
- EC2: Backend + Judge workers
- RDS: PostgreSQL (multi-AZ)
- ElastiCache: Redis
- ALB: Load balancer
- S3: File storage
- CloudFront: CDN
- Route53: DNS
```

### Option 2: DigitalOcean
```
- Droplets: Backend, Judge, Database
- Managed Database: PostgreSQL
- App Platform: Deploy directly
- Spaces: Object storage
- Load Balancer
```

### Option 3: Self-Hosted
```
- Proxmox/KVM: Virtual machines
- Nginx: Load balancer
- PostgreSQL: Database
- Redis: Cache
- Docker: Containerization
```

## Docker Production Setup

### docker-compose.prod.yml
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: o1_peoples_nginx
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - backend
    networks:
      - o1_peoples_network

  backend:
    image: o1peoples/backend:latest
    container_name: o1_peoples_backend
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://...
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
    depends_on:
      - postgres
      - redis
    networks:
      - o1_peoples_network

  postgres:
    image: postgres:16
    container_name: o1_peoples_db
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - o1_peoples_network

  redis:
    image: redis:7-alpine
    container_name: o1_peoples_redis
    volumes:
      - redis_data:/data
    networks:
      - o1_peoples_network

  judge_worker:
    image: o1peoples/judge:latest
    deploy:
      replicas: 4
    depends_on:
      - redis
    networks:
      - o1_peoples_network

volumes:
  postgres_data:
  redis_data:

networks:
  o1_peoples_network:
    driver: bridge
```

## SSL/TLS Setup

### Let's Encrypt (Free)
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot certonly --standalone -d o1peoples.com

# Auto-renew
sudo systemctl enable certbot.timer
```

### Nginx SSL Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name o1peoples.com;

    ssl_certificate /etc/letsencrypt/live/o1peoples.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/o1peoples.com/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

server {
    listen 80;
    server_name o1peoples.com;
    return 301 https://$server_name$request_uri;
}
```

## Environment Variables

### .env.production
```bash
# === APPLICATION ===
NODE_ENV=production
PORT=3000
JWT_SECRET=<generate-strong-secret>

# === DATABASE ===
DATABASE_URL=postgresql://user:password@db.example.com:5432/o1_peoples_db
DB_POOL_MIN=10
DB_POOL_MAX=30

# === REDIS ===
REDIS_URL=redis://redis.example.com:6379

# === SECURITY ===
CORS_ORIGIN=https://o1peoples.com
RATE_LIMIT_LOGIN_PER_MIN=5
RATE_LIMIT_SUBMIT_PER_MIN=12

# === MONITORING ===
LOG_LEVEL=info
SENTRY_DSN=<sentry-dsn>

# === FILE STORAGE ===
STORAGE_PATH=/data/storage
MAX_FILE_SIZE=10485760
```

## Monitoring & Observability

### Prometheus Setup
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'backend'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'postgres'
    static_configs:
      - targets: ['localhost:9187']
  
  - job_name: 'redis'
    static_configs:
      - targets: ['localhost:9121']
```

### Grafana Dashboards
- Backend metrics
- Database performance
- Redis usage
- Judge queue
- User activity

### Alerting
```yaml
groups:
  - name: production
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m

      - alert: DatabaseDown
        expr: up{job="postgres"} == 0
        for: 1m

      - alert: QueueBacklog
        expr: queue_length > 1000
        for: 5m
```

## Backup Strategy

### Database Backups
```bash
# Automated daily backups
0 2 * * * pg_dump -U postgres -d o1_peoples_db > /backups/$(date +\%Y\%m\%d).sql

# Off-site backup to S3
0 3 * * * aws s3 cp /backups/*.sql s3://o1-backups/
```

### File Storage Backups
```bash
# Backup problem files
0 4 * * * tar -czf /backups/storage-$(date +\%Y\%m\%d).tar.gz /data/storage/

# Upload to S3
aws s3 sync /backups/ s3://o1-backups/
```

### Recovery Test
```bash
# Monthly: Test recovery from backup
# Ensure backups are valid and recoverable
```

## CI/CD Pipeline (GitHub Actions)

### .github/workflows/deploy.yml
```yaml
name: Deploy Production

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build Docker image
        run: docker build -t o1peoples/backend:latest backend/
      
      - name: Push to registry
        run: docker push o1peoples/backend:latest
      
      - name: Deploy to server
        run: |
          ssh deploy@server "cd /app && docker-compose pull && docker-compose up -d"
```

## Deployment Checklist

### Pre-Deployment
- [ ] All tests passing
- [ ] Load tests successful
- [ ] Database migrated
- [ ] Backup taken
- [ ] SSL certificates renewed
- [ ] Secrets configured
- [ ] Monitoring setup

### Deployment
- [ ] Pull latest code
- [ ] Build Docker images
- [ ] Push to registry
- [ ] Update docker-compose
- [ ] Start services
- [ ] Verify health checks
- [ ] Test API endpoints
- [ ] Check logs

### Post-Deployment
- [ ] Monitor metrics
- [ ] Check error logs
- [ ] Verify database
- [ ] Test critical flows
- [ ] Confirm backups

## Scaling Strategy

### Horizontal Scaling
```
Initial: 1 Backend, 1 Database, 4 Judge Workers
Phase 1: 3 Backends, 1 Database (replica), 8 Judge Workers
Phase 2: 5 Backends, 2 Database nodes (cluster), 12 Workers
Phase 3: K8s cluster with auto-scaling
```

### Database Scaling
```
Write: Master
Read: 2-3 replicas
Replication lag: < 100ms
```

### Judge Scaling
```
Initial: 4 workers
Growth: 8 → 12 → 16+
Auto-scale based on queue length
```

## Cost Optimization

### Resource Usage
```
Backend: $50-100/month
Database: $30-50/month
Redis: $15-25/month
Storage: $10-20/month
CDN: $5-15/month
Total: ~$110-210/month for 400 users
```

### Optimization Options
- Spot instances (25-70% discount)
- Reserved instances (30-40% discount)
- Auto-scaling during off-peak
- Compress/archive old data

## Disaster Recovery

### RTO: 1 hour (Recovery Time Objective)
### RPO: 15 minutes (Recovery Point Objective)

```
Plan:
1. Database replication (multi-region)
2. Automated backups (hourly)
3. Off-site backup storage
4. Documented recovery procedures
5. Quarterly DR tests
```

## Security Hardening

```
✅ HTTPS/TLS
✅ CORS properly configured
✅ Rate limiting enabled
✅ Input validation
✅ SQL injection prevention (Prisma)
✅ XSS protection
✅ CSRF protection
✅ Secrets management
✅ Database encryption
✅ Regular security updates
```

## Go-Live Checklist

- [ ] All features implemented
- [ ] Tests passing (100%)
- [ ] Load tests successful
- [ ] Security audit complete
- [ ] Monitoring/alerting setup
- [ ] Documentation complete
- [ ] Team trained
- [ ] Backup procedure tested
- [ ] Rollback plan ready
- [ ] Support team ready

---

## Post-Launch

### Week 1-2: Monitor
- 24/7 support
- Daily health checks
- Performance monitoring
- Bug fixes

### Week 3-4: Optimize
- Tune database queries
- Optimize caches
- Analyze user behavior
- Fix reported issues

### Month 2+: Iterate
- Feature requests
- User feedback
- Performance improvements
- Scaling as needed

---

**Deployment Complete! 🚀**
