# Agent Escrow Backend

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-20.x-green)](https://nodejs.org/)
[![NestJS](https://img.shields.io/badge/NestJS-10.x-red)](https://nestjs.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)](https://www.typescriptlang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15.x-blue)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-7.x-red)](https://redis.io/)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen)]()
[![Coverage](https://img.shields.io/badge/coverage-85%25-yellowgreen)]()

## 📋 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Database Schema](#database-schema)
- [Environment Variables](#environment-variables)
- [Development](#development)
- [Testing](#testing)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

Agent Escrow Backend is a production-ready NestJS application that powers the Agent Escrow platform. It provides REST APIs for:

- **Agent Management** - Create, read, update, pause, and resume AI agents
- **Payment Processing** - Execute and track agent payments
- **Spending Limits** - Configure daily, weekly, and monthly limits
- **Transaction History** - Complete audit trail of all payments
- **WebSocket Updates** - Real-time notifications for payment status

## 🛠 Tech Stack

| Category | Technology | Version |
|----------|------------|---------|
| Runtime | Node.js | 20.x |
| Framework | NestJS | 10.x |
| Language | TypeScript | 5.x |
| Database | PostgreSQL | 15.x |
| ORM | TypeORM | 0.3.x |
| Cache | Redis | 7.x |
| Queue | BullMQ | 4.x |
| WebSocket | Socket.IO | 4.x |
| Validation | class-validator | 0.14.x |
| Testing | Jest | 29.x |
| Logging | Winston | 3.x |

## 🏗 Architecture
┌─────────────────────────────────────────────────────────────────┐
│ Client Applications │
│ (Web Dashboard, Mobile App, External Services) │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ API Gateway (Port 3001) │
│ REST API + WebSocket │
└─────────────────────────────────────────────────────────────────┘
│
┌────────────────────────┼────────────────────────┐
▼ ▼ ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Agents │ │ Payments │ │ Users │
│ Module │ │ Module │ │ Module │
└──────────────┘ └──────────────┘ └──────────────┘
│ │ │
└────────────────────────┼────────────────────────┘
▼
┌─────────────────────────────────────────────────────────────────┐
│ Data Layer │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │ PostgreSQL │ │ Redis │ │ BullMQ │ │
│ │ (Primary) │ │ (Cache) │ │ (Queue) │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────────────┘

## 🚀 Getting Started

### Prerequisites

| Requirement | Version | Installation |
|-------------|---------|--------------|
| Node.js | 20.x+ | [nodejs.org](https://nodejs.org/) |
| npm | 9.x+ | Included with Node.js |
| PostgreSQL | 15.x+ | [postgresql.org](https://www.postgresql.org/) |
| Redis | 7.x+ | [redis.io](https://redis.io/) |

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/AgentPayEscrow/agent-escrow-indexer.git
cd agent-escrow-indexer

# 2. Install dependencies
npm install

# 3. Copy environment variables
cp .env.example .env

# 4. Edit .env with your database credentials
nano .env

# 5. Run database migrations
npm run migrate

# 6. Start the development server
npm run start:dev
📚 API Documentation
Once running, interactive API documentation is available at:

text
http://localhost:3001/api/docs
API Endpoints
Agents
Method	Endpoint	Description	Auth
GET	/api/v1/agents	List all agents	JWT
GET	/api/v1/agents/:id	Get agent by ID	JWT
POST	/api/v1/agents	Create new agent	JWT
PATCH	/api/v1/agents/:id	Update agent	JWT
DELETE	/api/v1/agents/:id	Delete agent	JWT
POST	/api/v1/agents/:id/pause	Pause agent	JWT
POST	/api/v1/agents/:id/resume	Resume agent	JWT
Payments
Method	Endpoint	Description	Auth
POST	/api/v1/payments	Execute payment	JWT
GET	/api/v1/payments/:id	Get payment by ID	JWT
GET	/api/v1/payments/agent/:agentId	Get agent payments	JWT
Limits
Method	Endpoint	Description	Auth
GET	/api/v1/agents/:id/limits	Get spending limits	JWT
PUT	/api/v1/agents/:id/limits	Update spending limits	JWT
Example Requests
Create Agent

bash
curl -X POST http://localhost:3001/api/v1/agents \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "address": "GABC123XYZ789",
    "name": "Trading Bot",
    "dailyLimit": 1000,
    "weeklyLimit": 5000,
    "monthlyLimit": 20000
  }'
Execute Payment

bash
curl -X POST http://localhost:3001/api/v1/payments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "agentId": 1,
    "toAddress": "GXYZ987UVW654",
    "amount": 50000,
    "memo": "API payment"
  }'
🗄 Database Schema
Agents Table
Column	Type	Description
id	SERIAL	Primary key
agent_id	INTEGER	Unique agent identifier
address	VARCHAR(56)	Stellar wallet address
name	VARCHAR(100)	Agent display name
status	VARCHAR(20)	active, paused, terminated
total_deposited	BIGINT	Total funds deposited
total_spent	BIGINT	Total funds spent
remaining_balance	BIGINT	Current available balance
spending_limits	JSONB	Daily/weekly/monthly limits
created_at	TIMESTAMP	Creation timestamp
updated_at	TIMESTAMP	Last update timestamp
Transactions Table
Column	Type	Description
id	SERIAL	Primary key
transaction_id	INTEGER	Unique transaction identifier
agent_id	INTEGER	Foreign key to agents
from_address	VARCHAR(56)	Sender address
to_address	VARCHAR(56)	Recipient address
token_address	VARCHAR(56)	Token contract address
amount	BIGINT	Payment amount
fee	BIGINT	Platform fee
net_amount	BIGINT	Amount after fee
status	VARCHAR(20)	pending, executed, failed
memo	TEXT	Optional payment memo
created_at	TIMESTAMP	Transaction timestamp
🔧 Environment Variables
Variable	Required	Default	Description
NODE_ENV	No	development	Environment (development/production)
PORT	No	3001	Server port
DB_HOST	Yes	localhost	PostgreSQL host
DB_PORT	No	5432	PostgreSQL port
DB_USERNAME	Yes	postgres	Database username
DB_PASSWORD	Yes	-	Database password
DB_DATABASE	Yes	agent_escrow	Database name
REDIS_HOST	No	localhost	Redis host
REDIS_PORT	No	6379	Redis port
JWT_SECRET	Yes	-	JWT signing secret
JWT_EXPIRATION	No	7d	JWT expiration time
💻 Development
Available Scripts
Command	Description
npm run start:dev	Start with hot reload
npm run build	Build for production
npm run start:prod	Run production build
npm run lint	Run ESLint
npm run format	Format with Prettier
npm run test	Run unit tests
npm run test:cov	Run tests with coverage
npm run test:e2e	Run end-to-end tests
npm run migrate	Run database migrations
npm run seed	Seed database
npm run swagger	Generate Swagger docs
Code Structure
text
src/
├── main.ts                 # Application entry point
├── app.module.ts           # Root module
├── common/                 # Shared utilities
│   ├── decorators/         # Custom decorators
│   ├── filters/            # Exception filters
│   ├── guards/             # Auth guards
│   ├── interceptors/       # Request interceptors
│   ├── pipes/              # Validation pipes
│   └── utils/              # Helper functions
├── config/                 # Configuration
├── database/               # Database setup
│   ├── migrations/         # Migration files
│   ├── seeds/              # Seed data
│   └── entities/           # TypeORM entities
└── modules/                # Feature modules
    ├── agents/             # Agent management
    ├── payments/           # Payment processing
    ├── auth/               # Authentication
    └── users/              # User management
🧪 Testing
Unit Tests
bash
# Run all unit tests
npm run test

# Run with coverage
npm run test:cov

# Run in watch mode
npm run test:watch
E2E Tests
bash
# Run end-to-end tests
npm run test:e2e
Test Coverage Requirements
Module	Minimum Coverage
Agents	85%
Payments	90%
Auth	80%
Users	85%
🐳 Docker
Build Image
bash
docker build -t agent-escrow-backend .
Run with Docker Compose
bash
docker-compose up -d
Docker Compose Configuration
yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: agent_escrow
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build: .
    ports:
      - "3001:3001"
    depends_on:
      - postgres
      - redis
    environment:
      DB_HOST: postgres
      REDIS_HOST: redis

volumes:
  postgres_data:
🚢 Deployment
Production Checklist
Set NODE_ENV=production

Use strong JWT_SECRET

Enable HTTPS

Configure rate limiting

Set up monitoring

Configure log aggregation

Set up database backups

Deployment Options
Platform	Guide
AWS	Elastic Beanstalk / ECS
Google Cloud	Cloud Run
Heroku	Heroku Deploy
DigitalOcean	App Platform
Render	Render Deploy
🤝 Contributing
Fork the repository

Create a feature branch

Make your changes

Run tests

Submit a pull request

See CONTRIBUTING.md for detailed guidelines.

📄 License
This project is licensed under the MIT License - see the LICENSE file for details.

🙏 Acknowledgments
Stellar Development Foundation

Drips Wave Program

Open Source Community

Built with ❤️ for the Stellar Ecosystem
