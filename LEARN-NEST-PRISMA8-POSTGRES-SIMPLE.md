# LEARN-NEST-PRISMA8-POSTGRES-SIMPLE.md

A simple, step-by-step learning guide for **NestJS (REST API)** + **Prisma 8** + **PostgreSQL (Local)**. Self-paced with lessons. 

> Focus: NestJS syntax, simple CRUD → Auth (JWT) → Auth0, no Docker, simple ports (Nest: `3000`, Postgres: `5432`), `npm` only, no testing, deployment included. Windows CMD and PowerShell commands are given.

---

## Table of Contents

- [0. How To Use This Guide](#0-how-to-use-this-guide)
- [1. Prerequisites](#1-prerequisites)
- [2. NestJS Basics (Syntax + Logic)](#2-nestjs-basics-syntax--logic)
- [Lesson 1: Setup & Create Your First Nest App](#lesson-1-setup--create-your-first-nest-app)
- [Lesson 2: Simple REST "Hello" Endpoint](#lesson-2-simple-rest-hello-endpoint)
- [Lesson 3: Local PostgreSQL Setup (No Docker)](#lesson-3-local-postgresql-setup-no-docker)
- [Lesson 4: Prisma 8 Setup, Schema & Migrations](#lesson-4-prisma-8-setup-schema--migrations)
- [Lesson 5: Simple CRUD API (Nest + Prisma)](#lesson-5-simple-crud-api-nest--prisma)
- [Lesson 6: DTOs & Simple Validation](#lesson-6-dtos--simple-validation)
- [Lesson 7: Auth - JWT, bcrypt, Guards (Simple)](#lesson-7-auth---jwt-bcrypt-guards-simple)
- [Lesson 8: Auth0 Integration (Resource Server - Simple)](#lesson-8-auth0-integration-resource-server---simple)
- [Lesson 9: Relations, Ownership & Simple Pagination](#lesson-9-relations-ownership--simple-pagination)
- [Lesson 10: Deployment (Neon + Render/Railway) - Simple](#lesson-10-deployment-neon--renderrailway---simple)
- [Appendix A: Official Docs & References](#appendix-a-official-docs--references)
- [Appendix B: Command Reference (Windows CMD & PowerShell)](#appendix-b-command-reference-windows-cmd--powershell)
- [Appendix C: Simple Tips & Troubleshooting](#appendix-c-simple-tips--troubleshooting)

---

## 0. How To Use This Guide

- **Self-paced**: Do one lesson at a time. Don't jump ahead until the exercise works.
- **Flow**: Read → Code → Run → Verify. Do it manually (no automated tests here).
- **Focus on syntax**: Pay attention to decorators (`@Module`, `@Controller`, `@Injectable`, `@Get`, `@Post`, etc.) and why they're used.
- **Simple ports**: Nest runs on `http://localhost:3000`. PostgreSQL on `5432`.
- **npm only**: Use `npm` for all installs.
- **CMD and PowerShell**: Both commands are provided per step. Use whichever you prefer.
- **Logic blocks**: Short explanations of *why* things work, kept simple.

---

## 1. Prerequisites

Install these before starting.

| Tool | Version | Why |
|---|---|---|
| Node.js | 18 LTS or 20 LTS | Runs NestJS and Prisma |
| npm | comes with Node | Package manager (only one we use) |
| Git | latest | Version control |
| VS Code | latest | Editor |
| PostgreSQL | 14, 15, 16 or 17 | Database (installed locally, no Docker) |
| pgAdmin (optional) | latest | GUI to see your tables/data |

### Check versions

**CMD**
```cmd
node -v
npm -v
git --version
psql --version
```

**PowerShell**
```powershell
node -v
npm -v
git --version
psql --version
```

### Install Nest CLI globally

**CMD**
```cmd
npm install -g @nestjs/cli
nest --version
```

**PowerShell**
```powershell
npm install -g @nestjs/cli
nest --version
```

**Logic:** The Nest CLI (`nest new`, `nest g ...`) generates boilerplate files for you. Installing it globally means you can run `nest` from any folder.

**Exercise:** Run `nest --version`. You should see a version number.

---

## 2. NestJS Basics (Syntax + Logic)

Before writing code, understand the 4 building blocks. This is the syntax you must master.

### 2.1 Module — `@Module()`

A module groups related code. Every app has at least one root module (`AppModule`).

```typescript
import { Module } from '@nestjs/common';

@Module({
  imports: [],      // other modules this module needs
  controllers: [],  // HTTP endpoints (routes)
  providers: [],   // services / logic / injectable classes
  exports: [],      // what this module shares with other modules
})
export class AppModule {}
```

**Logic:** Nest uses modules for dependency injection. If `TodosService` lives in `TodosModule` and `AuthModule` needs it, `AuthModule` must `imports: [TodosModule]` and `TodosModule` must `exports: [TodosService]`.

### 2.2 Controller — `@Controller('path')`

Handles HTTP requests (REST). It should stay thin: receive, delegate, return.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('todos')     // route prefix -> /todos
export class TodosController {
  @Get()                // GET /todos
  findAll() {
    return { message: 'list of todos' };
  }

  @Get(':id')           // GET /todos/1
  findOne() {
    return { message: 'one todo' };
  }
}
```

HTTP decorators you will use:

| Decorator | Route | Purpose |
|---|---|---|
| `@Get()` | `GET /todos` | read |
| `@Post()` | `POST /todos` | create |
| `@Patch(':id')` | `PATCH /todos/:id` | partial update |
| `@Delete(':id')` | `DELETE /todos/:id` | delete |
| `@Param('id')` | — | read a route param |
| `@Body()` | — | read the JSON body |
| `@Query()` | — | read query string (`?limit=10`) |

**Logic:** In REST, a resource is a URL (`/todos`), and the HTTP verb says what to do. `GET` = read, `POST` = create, `PATCH` = update, `DELETE` = remove.

### 2.3 Service — `@Injectable()`

Contains business logic and database access. Services are injected into controllers.

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class TodosService {
  findAll() {
    return [{ id: 1, title: 'learn nest' }];
  }
}
```

Injecting a service into a controller:

```typescript
import { Controller, Get } from '@nestjs/common';
import { TodosService } from './todos.service';

@Controller('todos')
export class TodosController {
  constructor(private readonly todosService: TodosService) {}

  @Get()
  findAll() {
    return this.todosService.findAll();
  }
}
```

**Logic:** Dependency Injection (DI) means you do not `new TodosService()` yourself. You list it in `constructor`, Nest creates it and passes it in. Benefits: the service can itself have dependencies (like `PrismaService`), and you can swap the implementation later without touching the controller.

### 2.4 The request lifecycle (memorize this order)

```text
HTTP Request
   -> Middleware
   -> Guards          (auth? allow/deny)
   -> Interceptors    (pre-handler)
   -> Pipes           (validate + transform input)
   -> Controller      (handle route)
   -> Service         (business logic + DB)
   -> Interceptors    (post-handler: transform response)
   -> HTTP Response
```

**Logic:** Each piece runs before the next. Guards decide *whether* the request continues. Pipes clean the *input*. Interceptors wrap the *whole* request (great for logging and response shaping).

---

## Lesson 1: Setup & Create Your First Nest App

**Time:** ~15 minutes
**Goal:** Generate a running NestJS REST app on port 3000.

### Step 1.1 — Create the project

Go to your learning folder.

**CMD**
```cmd
cd C:\Users\sover\OneDrive\Desktop\learning\nest-learning
nest new simple-api
cd simple-api
npm run start:dev
```

**PowerShell**
```powershell
cd C:\Users\sover\OneDrive\Desktop\learning\nest-learning
nest new simple-api
cd simple-api
npm run start:dev
```

**Logic:** `nest new simple-api` generates a TypeScript REST project. `--strict` is on by default, which is good. `npm run start:dev` runs the app in watch mode: it restarts automatically when you save a file.

### Step 1.2 — Confirm it works

Open in browser: `http://localhost:3000`

You should see:

```json
{"message":"Hello World!"}
```

**Logic:** The generated `AppController` has a root `@Get()` route, so `GET /` responds. Port 3000 is Nest's default.

### Step 1.3 — Know the generated files

```text
simple-api/
  src/
    main.ts                 <- bootstrap file, where the app starts
    app.module.ts           <- root module
    app.controller.ts       <- root route
    app.service.ts          <- root service
  test/                     <- ignore (we skip testing)
  .env                      <- environment variables
  nest-cli.json             <- CLI config
  tsconfig.json             <- TypeScript config
  package.json              <- dependencies + scripts
```

`src/main.ts` (important — this is where the app is created):

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

**Logic:** `main.ts` creates the app from `AppModule` and starts listening. Everything else is just modules attached to `AppModule`.

### Step 1.4 — Useful scripts (from package.json)

```bash
npm run start        # normal start
npm run start:dev    # watch mode (use this while learning)
npm run start:prod   # production (after npm run build)
npm run build        # compile TypeScript to dist/
```

### Step 1.5 — Change the port (optional)

In `src/main.ts`:

```typescript
await app.listen(3000);          // default
await app.listen(process.env.PORT ?? 3000);   // flexible
```

**Logic:** Reading the port from an env var lets the same code run on any port in production. We will use this in deployment.

**Exercise:** Change the message in `AppController` to `{"message":"Nest is ready"}` and reload the browser.

---

## Lesson 2: Simple REST "Hello" Endpoint

**Time:** ~20 minutes
**Goal:** Practice NestJS syntax — module, controller, service, routes, params, body, query.

### Step 2.1 — Generate a module

**CMD / PowerShell (same command)**
```bash
nest g module cats
nest g service cats
nest g controller cats
```

**CMD**
```cmd
nest g module cats
nest g service cats
nest g controller cats
```

**PowerShell**
```powershell
nest g module cats
nest g service cats
nest g controller cats
```

`nest g` = `nest generate`. It creates files **and** wires them into the module for you.

Generated files:

```text
src/cats/
  cats.module.ts
  cats.controller.ts
  cats.service.ts
```

**Verify** `src/cats/cats.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

**Logic:** `nest g` auto-registered `CatsController` and `CatsService`. If you create files by hand, you must add them here yourself or Nest will not know about them.

### Step 2.2 — Register the module in AppModule

`src/app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Logic:** `imports: [CatsModule]` attaches the cats feature. This is how Nest composes an app: root module + feature modules.

### Step 2.3 — Write the service with in-memory data

`src/cats/cats.service.ts`:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';

@Injectable()
export class CatsService {
  private cats = [
    { id: 1, name: 'Tom', age: 3 },
    { id: 2, name: 'Jerry', age: 2 },
  ];

  findAll() {
    return this.cats;
  }

  findOne(id: number) {
    const cat = this.cats.find((c) => c.id === id);
    if (!cat) {
      throw new NotFoundException(`Cat with id ${id} not found`);
    }
    return cat;
  }

  create(input: { name: string; age: number }) {
    const cat = { id: this.cats.length + 1, ...input };
    this.cats.push(cat);
    return cat;
  }

  update(id: number, input: { name?: string; age?: number }) {
    const cat = this.findOne(id);
    Object.assign(cat, input);
    return cat;
  }

  remove(id: number) {
    const index = this.cats.findIndex((c) => c.id === id);
    if (index === -1) {
      throw new NotFoundException(`Cat with id ${id} not found`);
    }
    return this.cats.splice(index, 1);
  }
}
```

**Logic:**
- Data lives in the service, not the controller. If you restart the app, data is gone — that is fine for this lesson. A database comes in Lesson 5.
- `NotFoundException` makes Nest return HTTP **404** with `{ statusCode: 404, message: "..." }` automatically. Throwing Nest's built-in exceptions is the correct way to return error codes.
- `update()` calls `findOne(id)` first, so a missing cat gives **404** before anything is changed. `Object.assign` copies only the keys present in `input`, which is what makes `PATCH` a *partial* update.

### Step 2.4 — Write the controller with all 4 REST verbs

`src/cats/cats.controller.ts`:

```typescript
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  ParseIntPipe,
  Patch,
  Post,
} from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Get()
  findAll() {
    return this.catsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.catsService.findOne(id);
  }

  @Post()
  create(@Body() body: { name: string; age: number }) {
    return this.catsService.create(body);
  }

  @Patch(':id')
  update(@Param('id', ParseIntPipe) id: number, @Body() body: { name?: string; age?: number }) {
    return this.catsService.update(id, body);
  }

  @Delete(':id')
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.catsService.remove(id);
  }
}
```

**Logic:**
- `private readonly catsService: CatsService` — DI. Nest injects the service instance.
- `@Param('id', ParseIntPipe)` — route params are **strings** (`"1"`). `ParseIntPipe` converts to number and returns **400** if it is not a number. Without it, arithmetic would break.
- `@Body()` — the parsed JSON body, available on POST/PATCH/PUT.
- The controller is thin: it extracts input and calls the service. Business rules live in the service.

### Step 2.5 — Check `update` works

The service already has all four CRUD methods (see Step 2.3), so there is nothing to add here. Just prove it in Step 2.6:

```http
PATCH http://localhost:3000/cats/1
Content-Type: application/json

{
  "age": 4
}
```

You get back `{"id":1,"name":"Tom","age":4}` — only `age` changed, `name` was left alone. That is the point of `PATCH` plus `Object.assign`.

### Step 2.6 — Test all routes manually

Install the **REST Client** VS Code extension, or use curl.

**curl in CMD**
```cmd
curl http://localhost:3000/cats
curl http://localhost:3000/cats/1
curl -X POST http://localhost:3000/cats -H "Content-Type: application/json" -d "{\"name\":\"Kitty\",\"age\":1}"
curl -X PATCH http://localhost:3000/cats/1 -H "Content-Type: application/json" -d "{\"age\":4}"
curl -X DELETE http://localhost:3000/cats/2
```

**curl in PowerShell**
```powershell
curl http://localhost:3000/cats
curl http://localhost:3000/cats/1
Invoke-RestMethod -Uri http://localhost:3000/cats -Method POST -ContentType "application/json" -Body '{"name":"Kitty","age":1}'
Invoke-RestMethod -Uri http://localhost:3000/cats/1 -Method PATCH -ContentType "application/json" -Body '{"age":4}'
Invoke-RestMethod -Uri http://localhost:3000/cats/2 -Method DELETE
```

**REST Client** — create `requests.http`:

```http
GET http://localhost:3000/cats

###

GET http://localhost:3000/cats/1

###

POST http://localhost:3000/cats
Content-Type: application/json

{
  "name": "Kitty",
  "age": 1
}

###

PATCH http://localhost:3000/cats/1
Content-Type: application/json

{
  "age": 4
}

###

DELETE http://localhost:3000/cats/2
```

Also try `GET http://localhost:3000/cats/abc` — you get **400 Bad Request** from `ParseIntPipe`. This is validation working.

**Exercise:** Add a `GET /cats/search?name=Tom` route with `@Query('name')`.

---

## Lesson 3: Local PostgreSQL Setup (No Docker)

**Time:** ~30 minutes
**Goal:** Install PostgreSQL on Windows, create a database, and be able to query it. Port `5432`.

### Step 3.1 — Install PostgreSQL

**Option A: winget (easiest)**

**CMD**
```cmd
winget install PostgreSQL.PostgreSQL.16
```

**PowerShell**
```powershell
winget install PostgreSQL.PostgreSQL.16
```

**Option B: official installer**

Download from https://www.postgresql.org/download/windows/ and run the installer.

During install, set these (write them down):
- Port: **5432** (default, keep it)
- Superuser: **postgres**
- Password: choose something and remember it (example: `postgres`)
- Locale / Encoding: default

The installer also installs **pgAdmin 4**.

### Step 3.2 — Add `psql` to PATH

The installer may not add it. Find it at:

```text
C:\Program Files\PostgreSQL\16\bin\psql.exe
```

Add that `bin` folder to your Windows PATH (`Environment Variables` → `Path` → New), then open a **new** terminal.

### Step 3.3 — Verify the server is running

**CMD**
```cmd
psql --version
pg_isready -h localhost -p 5432
```

**PowerShell**
```powershell
psql --version
pg_isready -h localhost -p 5432
```

`pg_isready` should print `accepting connections`.

**Logic:** PostgreSQL runs as a background Windows service on port 5432. If `pg_isready` says `no response`, the service is stopped.

### Step 3.4 — Manage the service (CMD and PowerShell)

```cmd
net start postgresql-x64-16
net stop postgresql-x64-16
```

```powershell
Start-Service postgresql-x64-16
Stop-Service postgresql-x64-16
Get-Service postgresql*
```

**Logic:** `net start` / `Start-Service` are two ways to do the same thing. Use whichever you remember.

### Step 3.5 — Connect with `psql`

**CMD**
```cmd
psql -U postgres -h localhost -p 5432
```

**PowerShell**
```powershell
psql -U postgres -h localhost -p 5432
```

It asks for the password you chose during install. You land in the `postgres=#` prompt.

### Step 3.6 — Create your database

Inside `psql`:

```sql
CREATE DATABASE nestdb;
\l
\c nestdb
SELECT current_database(), current_user;
\dt
\q
```

| Command | Meaning |
|---|---|
| `\l` | list databases |
| `\c nestdb` | connect to a database |
| `\dt` | list tables in the current database |
| `\q` | quit psql |

**Logic:** PostgreSQL is a *server* with many databases. Each database has its own tables. Your app's connection string picks one database by name.

### Step 3.7 — Create a separate user (better than using postgres)

```sql
CREATE USER nestapp WITH PASSWORD 'nestpass';
GRANT ALL PRIVILEGES ON DATABASE nestdb TO nestapp;
\c nestdb
GRANT ALL ON SCHEMA public TO nestapp;
\q
```

Now your app has its own user. **Logic:** Never connect your app as the `postgres` superuser in real projects. If a bug drops a table, a normal user may not be able to.

### Step 3.8 — Your connection string

```text
postgresql://nestapp:nestpass@localhost:5432/nestdb
```

Format:

```text
postgresql://USERNAME:PASSWORD@HOST:PORT/DATABASE
```

**Logic:** Your app does not need a running GUI. It just needs this one string. NestJS + Prisma will connect with it.

### Step 3.9 — (Optional) pgAdmin

Open **pgAdmin 4** → **Servers** → **Add New Server**:
- Name: `local`
- Host: `localhost`
- Port: `5432`
- Maintenance DB: `postgres`
- Username: `postgres`

You get a visual table editor. Use it to peek at data, but learn SQL too.

**Exercise:** Create a table by hand and drop it.

```sql
\c nestdb
CREATE TABLE practice (id SERIAL PRIMARY KEY, name TEXT);
INSERT INTO practice (name) VALUES ('hello');
SELECT * FROM practice;
DROP TABLE practice;
```

`SERIAL` = auto-incrementing integer primary key. This is exactly what Prisma's `Int @id @default(autoincrement())` generates.

---

## Lesson 4: Prisma 8 Setup, Contract & Client

**Time:** ~40 minutes
**Goal:** Connect the Nest app to Postgres through Prisma 8.

### Step 4.0 — Read this first: Prisma 8 is a big rewrite

Prisma 8 is the newest line and it works differently from the older Prisma tutorials you will find online:

| Older Prisma (5/6/7) | Prisma 8 |
|---|---|
| `schema.prisma` | `contract.prisma` |
| `generator client { provider = "prisma-client-js" }` | no `generator` block at all |
| `datasource db { url = env("DATABASE_URL") }` | no `datasource` block, connection lives in `prisma.config.ts` |
| `npx prisma migrate dev` | `npx prisma contract emit` + `npx prisma db init` / `migration plan` + `db migrate` |
| `import { PrismaClient } from "@prisma/client"` | `import { db } from "./prisma/db"` |
| `prisma.todo.findMany()` | `db.orm.public.Todo.all()` |
| `prisma.todo.findUnique({ where })` | `db.orm.public.Todo.where({ ... }).first()` |
| `$transaction([...])` | `db.transaction(async (tx) => { ... })` |

The model syntax (`model`, fields, `@id`, `@unique`, `@relation`) is the same, so the concepts transfer. Only the plumbing changed.

Also: **Prisma 8 requires Node.js 22.18 or newer.** Check with `node -v` before continuing. And Prisma 8 is currently a **release candidate** (`8.0.0-rc.x`). If you hit a bug, Lesson 4.9 shows the Prisma 7 fallback, which is nearly identical.

### Step 4.1 — Install the packages

**CMD / PowerShell**
```bash
npm install @prisma/orm-postgres
npm install -D prisma
npm install -D dotenv
```

| Package | Why |
|---|---|
| `@prisma/orm-postgres` | the runtime your app imports (`postgres()` and the `db` client) |
| `prisma` | the CLI: `contract emit`, `db init`, `db migrate` |
| `dotenv` | loads your `.env` file (Prisma 8 does not load it by itself) |

**Logic:** Prisma is split into a CLI (build-time, dev dependency) and a runtime library (shipped with your app). The CLI is not needed in production.

### Step 4.2 — Initialize

**CMD**
```cmd
npx prisma orm init --write-env
```

**PowerShell**
```powershell
npx prisma orm init --write-env
```

It creates:

```text
prisma.config.ts                <- config: which contract, which database
src/prisma/contract.prisma      <- your models (the "data contract")
src/prisma/db.ts                <- the client you import everywhere
.env                            <- your DATABASE_URL
.env.example
```

### Step 4.3 — Put your connection string in `.env`

```env
DATABASE_URL="postgresql://nestapp:nestpass@localhost:5432/nestdb"
```

**Logic:** `postgresql://user:password@host:port/database`. Same string format as Lesson 3.

### Step 4.4 — Read `prisma.config.ts`

```typescript
import "dotenv/config";
import { definePrismaConfig } from "prisma/config";
import { defineConfig as ormConfig } from "@prisma/orm-postgres/config";

export default definePrismaConfig({
  orm: ormConfig({
    contract: "./src/prisma/contract.prisma",
    db: {
      connection: process.env["DATABASE_URL"]!,
    },
  }),
});
```

**Logic:**
- `import "dotenv/config"` reads your `.env` so `process.env.DATABASE_URL` exists.
- `contract` points at the file that describes your models.
- `db.connection` is the database.
- The `!` says "I know this is defined" — TypeScript thinks `process.env` values may be `undefined`.
- `@prisma/orm-postgres/config` is what selects PostgreSQL. There is no `provider` field anymore.

### Step 4.5 — Write your models in `contract.prisma`

The first line **must** be `// use prisma-8` (the init command already wrote it). Then your models:

```prisma
// use prisma-8

model Todo {
  id        String   @id @default(cuid(2))
  title     String
  completed Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

| Syntax | Meaning |
|---|---|
| `model Todo { ... }` | a table + a TypeScript type |
| `id String @id` | primary key |
| `@default(cuid(2))` | Prisma generates the id, you never pass it |
| `title String` | required text column (`TEXT NOT NULL`) |
| `completed Boolean @default(false)` | has a default value |
| `createdAt DateTime @default(now())` | filled by the database on insert |
| `updatedAt DateTime @updatedAt` | filled automatically on every update |
| `String?` | `?` = nullable column |

**Logic:** `cuid(2)` is a short, sortable, URL-safe ID. `@updatedAt` is Prisma's magic attribute — it saves you writing `new Date()` on every update.

### Step 4.6 — Emit the contract

**CMD**
```cmd
npx prisma contract emit
```

**PowerShell**
```powershell
npx prisma contract emit
```

Now `src/prisma/` contains:

```text
src/prisma/
  contract.prisma     <- you write this
  contract.json       <- generated (machine readable)
  contract.d.ts       <- generated (TypeScript types)
  db.ts               <- the client
```

**Logic:** Prisma 8 splits "what you write" from "what the code reads". `contract.prisma` is the source; `contract.json` + `contract.d.ts` are the compiled output. Your app never reads the `.prisma` file at runtime. **Run `contract emit` after every model change.**

### Step 4.7 — Read `db.ts` and fix the JSON import for NestJS

`orm init` writes something like:

```typescript
import "dotenv/config";
import postgres from "@prisma/orm-postgres/runtime";
import type { Contract } from "./contract.d";
import contractJson from "./contract.json" with { type: "json" };

export const db = postgres<Contract>({
  contractJson,
  url: process.env["DATABASE_URL"]!,
});
```

NestJS compiles to **CommonJS** by default, and `with { type: "json" }` is an ESM-only syntax. So:

**Step A** — edit `tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "resolveJsonModule": true
  }
}
```

Keep Nest's other options (`strict`, `emitDecoratorMetadata`, `experimentalDecorators`) exactly as they are.

**Step B** — in `db.ts`, drop the `with` part:

```typescript
import "dotenv/config";
import postgres from "@prisma/orm-postgres/runtime";
import type { Contract } from "./contract.d";
import contractJson from "./contract.json";

export const db = postgres<Contract>({
  contractJson,
  url: process.env["DATABASE_URL"]!,
});
```

**Logic:** With `module: "nodenext"` and no `"type": "module"` in `package.json`, TypeScript treats your app as CommonJS, and CommonJS JSON imports do not use the `with` attribute. `resolveJsonModule: true` is what allows importing a `.json` file at all.

Also add the generated folder to `.gitignore`? No — Prisma tells you to **commit** `contract.json` and `contract.d.ts`, so teammates and deploys do not need to run `emit` themselves. Add `src/prisma/contract.ts` style artifacts only if you author in TypeScript.

### Step 4.8 — Create the tables

**CMD**
```cmd
npx prisma db init
```

**PowerShell**
```powershell
npx prisma db init
```

Verify:

```cmd
npx prisma db verify
```

**Logic:** `db init` creates the tables in an **empty** database and records that the database now matches your contract. `db verify` is the health check — it compares your contract against the live database. Use it in CI/deploy.

Check in psql that the table exists:

```sql
\c nestdb
\d "Todo"
```

### Step 4.9 — The everyday loop (memorize this)

```text
1. Edit src/prisma/contract.prisma
2. npx prisma contract emit        <- regenerate types
3. npx prisma migration plan       <- write a migration, read it
4. npx prisma db migrate           <- apply it
5. npx prisma db verify            <- confirm
```

**CMD**
```cmd
npx prisma contract emit
npx prisma migration plan
npx prisma db migrate
npx prisma db verify
```

**PowerShell**
```powershell
npx prisma contract emit
npx prisma migration plan
npx prisma db migrate
npx prisma db verify
```

**Logic:** `migration plan` writes the SQL steps as a *file* so you can read them before they run. `db migrate` applies it. `db init` is only for a brand-new empty database — after that you use `plan` + `migrate` for every change.

### Step 4.10 — Quick test of the client

Add `src/prisma/debug.controller.ts`... actually simpler: test from a service later. If you want a fast check, run this one-off script.

Create `test-db.ts` in the project root:

```typescript
import { db } from "./src/prisma/db";

async function main() {
  const created = await db.orm.public.Todo.create({ title: "first todo" });
  console.log("created:", created);
  const all = await db.orm.public.Todo.all();
  console.log("all:", all);
  await db.close();
}

main();
```

Run it with `npx tsx test-db.ts` (install once: `npm i -D tsx`).

Expected output: the created row with a generated `id` and `createdAt`, then the array with that same row.

**Logic:** `db.orm` holds your models. `public` is the PostgreSQL schema name (almost always `public`). `.create({...})` passes fields directly — note there is **no `data:` wrapper** in Prisma 8. `.all()` runs the query and returns every row. `db.close()` releases the connection pool.

### Step 4.11 — Prisma 8 quick reference

| I want to... | Prisma 8 |
|---|---|
| all rows | `await db.orm.public.Todo.all()` |
| filter | `.where({ completed: true })` |
| one row or null | `.where({ id }).first()` |
| one row or error | `.where({ id }).firstOrThrow()` |
| create | `.create({ title: "x" })` |
| update one | `.where({ id }).update({ title: "y" })` |
| update all matches | `.where({...}).updateAll({...})` |
| delete one | `.where({ id }).delete()` |
| delete all matches | `.where({...}).deleteAll()` |
| count only | `.where({...}).updateAndCount({...})` / `.deleteAndCount(...)` |
| upsert | `.upsert({ create: {...}, update: {...}, conflictOn: { email } })` |
| select fields | `.select("id", "title").all()` |
| sort | `.orderBy("createdAt")` |
| paginate | `.limit(10).offset(10)` |
| include relation | `.include("posts").first()` |
| text search | `.where((p) => p.title.ilike("%nest%")).all()` |
| transaction | `await db.transaction(async (tx) => { ... tx.orm... })` |
| count rows | `.aggregate((a) => ({ n: a.count() }))` |
| raw SQL | `db.raw.sql\`SELECT ...\`` |

**Important:** `.update()` and `.delete()` change exactly **one** row even if the filter matches many. Use `updateAll` / `deleteAll` when you mean all of them. That is deliberate, so a careless filter cannot wipe a table.

### Step 4.12 — Fallback: use Prisma 7 instead

If you are not ready for a release candidate, use Prisma 7. It is stable, and 90% of your Prisma knowledge is identical.

**CMD**
```cmd
npm uninstall prisma @prisma/orm-postgres
npm install -D prisma@7
npm install @prisma/client@7 @prisma/adapter-pg@7
npx prisma init --datasource-provider postgresql
```

`schema.prisma`:

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Todo {
  id        String   @id @default(cuid(2))
  title     String
  completed Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

`prisma.config.ts`:

```typescript
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: { path: "prisma/migrations" },
  datasource: { url: env("DATABASE_URL") },
});
```

`src/prisma.service.ts`:

```typescript
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client";

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  constructor() {
    super({ adapter: new PrismaPg({ connectionString: process.env.DATABASE_URL! }) });
  }

  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

Prisma 7 commands: `npx prisma generate`, `npx prisma migrate dev --name init`, `npx prisma migrate deploy`, `npx prisma studio`.

**Logic:** Note the extra pieces Prisma 7 needs: a `generator` block with an `output`, a `datasource` block with `url`, and a **driver adapter** (`@prisma/adapter-pg`) because Prisma 7+ no longer bundles the Postgres driver. Prisma 8 removes all three. Same models, same concepts.

Everything from Lesson 5 onward works with either version — only the import line and the method names differ.

**Exercise:** Add a `User` model with `id`, `email @unique`, `name String?`, `createdAt`. Run `contract emit`, `migration plan`, `db migrate`, `db verify`. Confirm with `\d "User"` in psql.

---

## Lesson 5: Simple CRUD API (Nest + Prisma)

**Time:** ~60 minutes
**Goal:** A complete REST CRUD for `Todo`, backed by Postgres. This is your first real full-stack API.

### Step 5.1 — Generate the module

**CMD**
```cmd
nest g module todos
nest g service todos
nest g controller todos
nest g resource todos
```

**PowerShell**
```powershell
nest g module todos
nest g service todos
nest g controller todos
nest g resource todos
```

`nest g resource todos` generates the 5 files you always need: module, controller, service, entity (DTO), dto. Skip it if you already made them.

```text
src/todos/
  todos.module.ts
  todos.controller.ts
  todos.service.ts
  todo.entity.ts
  dto/
    create-todo.dto.ts
    update-todo.dto.ts
```

### Step 5.2 — Create a shared `PrismaModule`

You want one database client for the whole app, not one per module.

`src/prisma/prisma.module.ts`:

```typescript
import { Global, Module } from "@nestjs/common";
import { PrismaService } from "./prisma.service";

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

`src/prisma/prisma.service.ts` (Prisma 8):

```typescript
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
import { db } from "./db";

@Injectable()
export class PrismaService implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await db.connect();
  }

  async onModuleDestroy() {
    await db.close();
  }
}
```

Register it once in `src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { PrismaModule } from "./prisma/prisma.module";
import { TodosModule } from "./todos/todos.module";

@Module({
  imports: [PrismaModule, TodosModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Logic:** `@Global()` means you do not have to import `PrismaModule` into every feature module — it is available app-wide. `OnModuleInit` / `OnModuleDestroy` are Nest lifecycle hooks: the first runs at startup, the second at shutdown. Connecting early means a bad `DATABASE_URL` fails at boot, not on the first request. Closing cleanly matters because the client holds a connection pool.

If you use Prisma 7, use the `PrismaService` from Lesson 4.12 instead (it extends `PrismaClient`). Nothing else changes.

### Step 5.3 — The service (all database code here)

`src/todos/todos.service.ts`:

```typescript
import { Injectable, NotFoundException } from "@nestjs/common";
import { PrismaService } from "../prisma/prisma.service";
import { CreateTodoDto } from "./dto/create-todo.dto";
import { UpdateTodoDto } from "./dto/update-todo.dto";

@Injectable()
export class TodosService {
  constructor(private readonly prisma: PrismaService) {}

  findAll() {
    return this.prisma.orm.public.Todo.all();
  }

  async findOne(id: string) {
    const todo = await this.prisma.orm.public.Todo.where({ id }).first();
    if (!todo) {
      throw new NotFoundException(`Todo with id "${id}" not found`);
    }
    return todo;
  }

  create(dto: CreateTodoDto) {
    return this.prisma.orm.public.Todo.create({
      title: dto.title,
      completed: dto.completed ?? false,
    });
  }

  async update(id: string, dto: UpdateTodoDto) {
    await this.findOne(id); // throws 404 if missing
    return this.prisma.orm.public.Todo.where({ id }).update({
      title: dto.title,
      completed: dto.completed,
    });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.orm.public.Todo.where({ id }).delete();
    return { message: `Todo ${id} deleted` };
  }

  removeAll() {
    return this.prisma.orm.public.Todo.where({}).deleteAll();
  }
}
```

**Logic, line by line:**

- `constructor(private readonly prisma: PrismaService)` — DI. Nest hands you the shared client.
- `.all()` — every row.
- `.where({ id }).first()` — returns the row **or `null`**. Never assume it exists.
- `.first()` returns `null` rather than throwing, so the service turns `null` into `NotFoundException` and Nest sends **404**. This is the pattern: DB returns null → you decide the HTTP meaning.
- `.create({ title, completed })` — fields **directly**, no `data:` wrapper in Prisma 8.
- `update()` — call `findOne` first so a missing row gives a clean 404 instead of `null` in the response body.
- `.where({ id }).delete()` — deletes exactly one. `delete()` returns the deleted row or `null`.
- `.where({}).deleteAll()` — an **empty filter** matches everything, so `deleteAll()` clears the table. This is how you delete all; `delete()` alone refuses to compile without a filter.

### Step 5.4 — The controller (thin)

`src/todos/todos.controller.ts`:

```typescript
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  Patch,
  Post,
} from "@nestjs/common";
import { CreateTodoDto } from "./dto/create-todo.dto";
import { UpdateTodoDto } from "./dto/update-todo.dto";
import { TodosService } from "./todos.service";

@Controller("todos")
export class TodosController {
  constructor(private readonly todosService: TodosService) {}

  @Get()
  findAll() {
    return this.todosService.findAll();
  }

  @Get(":id")
  findOne(@Param("id") id: string) {
    return this.todosService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateTodoDto) {
    return this.todosService.create(dto);
  }

  @Patch(":id")
  update(@Param("id") id: string, @Body() dto: UpdateTodoDto) {
    return this.todosService.update(id, dto);
  }

  @Delete(":id")
  remove(@Param("id") id: string) {
    return this.todosService.remove(id);
  }

  @Delete()
  removeAll() {
    return this.todosService.removeAll();
  }
}
```

**Logic:** Route order matters. `@Get(":id")` is declared after `@Get()`, so `GET /todos` matches the list route. If you put `:id` first, `GET /todos` would try to parse `"todos"` as an id. When you add `GET /todos/stats` later, declare it **before** `@Get(':id')`.

### Step 5.5 — The module

`src/todos/todos.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { TodosController } from "./todos.controller";
import { TodosService } from "./todos.service";

@Module({
  controllers: [TodosController],
  providers: [TodosService],
})
export class TodosModule {}
```

**Logic:** `PrismaService` comes from the `@Global()` `PrismaModule`, so no `imports` needed here. `TodosService` is not exported because only the controller uses it. Export a service only when another module needs it.

### Step 5.6 — The DTOs (plain for now, validated in Lesson 6)

`src/todos/dto/create-todo.dto.ts`:

```typescript
export class CreateTodoDto {
  title: string;
  completed?: boolean;
}
```

`src/todos/dto/update-todo.dto.ts`:

```typescript
export class UpdateTodoDto {
  title?: string;
  completed?: boolean;
}
```

**Logic:** A DTO (Data Transfer Object) describes the shape of incoming data. `?` means optional, matching PATCH semantics: you only send what you want to change. DTOs are also where you add validation rules in Lesson 6.

### Step 5.7 — Restart and test

**CMD**
```cmd
npm run start:dev
```

**PowerShell**
```powershell
npm run start:dev
```

Create `requests.http` at the project root:

```http
### create
POST http://localhost:3000/todos
Content-Type: application/json

{ "title": "learn NestJS" }

### list all
GET http://localhost:3000/todos

### get one (paste the id from the create response)
GET http://localhost:3000/todos/PASTE_ID_HERE

### update
PATCH http://localhost:3000/todos/PASTE_ID_HERE
Content-Type: application/json

{ "completed": true }

### delete
DELETE http://localhost:3000/todos/PASTE_ID_HERE

### delete all
DELETE http://localhost:3000/todos
```

Or with curl:

**CMD**
```cmd
curl -X POST http://localhost:3000/todos -H "Content-Type: application/json" -d "{\"title\":\"learn NestJS\"}"
curl http://localhost:3000/todos
curl http://localhost:3000/todos/1
curl -X PATCH http://localhost:3000/todos/1 -H "Content-Type: application/json" -d "{\"completed\":true}"
curl -X DELETE http://localhost:3000/todos/1
```

**PowerShell**
```powershell
Invoke-RestMethod -Uri http://localhost:3000/todos -Method POST -ContentType "application/json" -Body '{"title":"learn NestJS"}'
Invoke-RestMethod -Uri http://localhost:3000/todos
Invoke-RestMethod -Uri http://localhost:3000/todos/1
Invoke-RestMethod -Uri http://localhost:3000/todos/1 -Method PATCH -ContentType "application/json" -Body '{"completed":true}'
Invoke-RestMethod -Uri http://localhost:3000/todos/1 -Method DELETE
```

Check psql: `SELECT * FROM "Todo";`

### Step 5.8 — What to do when Prisma finds nothing / errors

| Symptom | Cause | Fix |
|---|---|---|
| `Cannot find module './contract.json'` | no `resolveJsonModule`, or you skipped `contract emit` | run `npx prisma contract emit`, check `tsconfig.json` |
| `P1001 Can't reach database server` | Postgres not running, or wrong host/port | `pg_isready -h localhost -p 5432`, check `.env` |
| `P1000 authentication failed` | wrong user/password | re-check `CREATE USER` from Lesson 3.7 |
| `P2002 unique constraint failed` | duplicate value on a `@unique` field | expected — handle it (Lesson 6.5) |
| Table does not exist | you edited the contract but never ran `migration plan` + `db migrate` | run the full loop from Lesson 4.9 |
| `CONTRACT.MARKER_MISMATCH` warning | database was not migrated to match the contract | `npx prisma db migrate` |

### Step 5.9 — Transactions (why they matter)

Two writes that must both succeed:

```typescript
async createWithDefaults(title: string) {
  return this.prisma.db.transaction(async (tx) => {
    const todo = await tx.orm.public.Todo.create({ title, completed: false });
    await tx.orm.public.Tag.createAll([{ name: `${title}-a` }, { name: `${title}-b` }]);
    return todo;
  });
}
```

**Logic:** Without a transaction, if the second write fails you are left with half the data. Inside `db.transaction(...)`, all writes commit together or all roll back. Query through `tx`, not `db` — queries on `db` run outside the transaction. Prisma 8 has no `$transaction([...])` array form; the callback is the only way, and it lets one result feed the next call.

**Exercise:** Add `GET /todos/stats` (declared **before** `@Get(':id')`) returning `{ total, completed }` using `.aggregate()`. Hint: `.aggregate((a) => ({ total: a.count() }))`, and a second query with `.where({ completed: true }).aggregate((a) => ({ n: a.count() }))`.

---

## Lesson 6: DTOs, Validation, Pipes, Error Handling

**Time:** ~45 minutes
**Goal:** Reject bad input automatically, and turn errors into clean JSON responses.

### Step 6.1 — Install the validation libraries

**CMD / PowerShell**
```bash
npm install class-validator class-transformer
```

### Step 6.2 — Turn on the global `ValidationPipe`

`src/main.ts`:

```typescript
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,            // strip properties with no decorator
      forbidNonWhitelisted: true, // ...and 400 if the client sent them
      transform: true,             // convert the body into DTO class instances
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

**Logic:** A pipe runs between the guard and the controller and can transform/validate the input. `ValidationPipe` uses the decorators you put on your DTO. `whitelist: true` is the important security setting: it removes any field the client sends that your DTO does not declare, so nobody can sneak in `{"title":"x","isAdmin":true}` and have it reach your database.

`transform: true` also converts route params, so a DTO with `@Type(() => Number) page: number` receives a real number.

### Step 6.3 — Add validation decorators to your DTOs

`src/todos/dto/create-todo.dto.ts`:

```typescript
import { IsBoolean, IsOptional, IsString, MaxLength, MinLength } from "class-validator";

export class CreateTodoDto {
  @IsString()
  @MinLength(3)
  @MaxLength(100)
  title: string;

  @IsOptional()
  @IsBoolean()
  completed?: boolean;
}
```

`src/todos/dto/update-todo.dto.ts`:

```typescript
import { IsBoolean, IsOptional, IsString, MaxLength, MinLength } from "class-validator";

export class UpdateTodoDto {
  @IsOptional()
  @IsString()
  @MinLength(3)
  @MaxLength(100)
  title?: string;

  @IsOptional()
  @IsBoolean()
  completed?: boolean;
}
```

Most used validators:

| Decorator | Rule |
|---|---|
| `@IsString()` | must be a string |
| `@IsInt()` / `@IsNumber()` | must be a number |
| `@IsBoolean()` | must be `true` or `false` |
| `@IsEmail()` | must look like an email |
| `@MinLength(n)` / `@MaxLength(n)` | string length |
| `@Min(n)` / `@Max(n)` | numeric range |
| `@IsOptional()` | may be absent |
| `@IsArray()` | must be an array |
| `@IsUUID()` | must be a UUID |
| `@Type(() => Number)` | convert the value (needs `transform: true`) |
| `@Transform(...)` | custom transformation |

Now test:

**CMD**
```cmd
curl -X POST http://localhost:3000/todos -H "Content-Type: application/json" -d "{\"title\":\"a\"}"
curl -X POST http://localhost:3000/todos -H "Content-Type: application/json" -d "{\"title\":\"valid title\",\"hacker\":true}"
```

**PowerShell**
```powershell
Invoke-RestMethod -Uri http://localhost:3000/todos -Method POST -ContentType "application/json" -Body '{"title":"a"}'
Invoke-RestMethod -Uri http://localhost:3000/todos -Method POST -ContentType "application/json" -Body '{"title":"valid title","hacker":true}'
```

Both return **400 Bad Request** with a list of which rule failed. Nothing reaches the database.

### Step 6.4 — Validate route params too

Create `src/todos/dto/todo-param.dto.ts`:

```typescript
import { IsUUID } from "class-validator";

export class TodoParamDto {
  @IsUUID()
  id: string;
}
```

Controller:

```typescript
@Get(":id")
findOne(@Param() params: TodoParamDto) {
  return this.todosService.findOne(params.id);
}
```

**Logic:** `cuid(2)` ids are not UUIDs, so for `cuid` ids use `@Matches(/^c[a-z0-9]{20,}$/)` or just `@IsString()`. If you switch the model's id to `@default(uuid())`, use `@IsUUID()`.

### Step 6.5 — Nest's built-in HTTP exceptions

```typescript
import {
  BadRequestException,
  ConflictException,
  ForbiddenException,
  NotFoundException,
  UnauthorizedException,
} from "@nestjs/common";

throw new NotFoundException("Todo not found");        // 404
throw new BadRequestException("Title too short");      // 400
throw new UnauthorizedException("Invalid credentials"); // 401
throw new ForbiddenException("Not your todo");          // 403
throw new ConflictException("Email already used");     // 409
```

Nest turns each into a consistent JSON body:

```json
{ "statusCode": 404, "message": "Todo not found", "error": "Not Found" }
```

Handling a unique-constraint violation from Prisma:

```typescript
async create(dto: CreateTodoDto) {
  try {
    return await this.prisma.orm.public.User.create({
      email: dto.email,
      passwordHash: dto.passwordHash,
    });
  } catch (error) {
    // 23505 = unique_violation in PostgreSQL
    if ((error as { sqlState?: string }).sqlState === "23505") {
      throw new ConflictException("That email is already registered");
    }
    throw error;
  }
}
```

**Logic:** The database is the last line of defence for uniqueness; your code should turn that into a friendly 409 instead of a 500.

### Step 6.6 — A global exception filter

`src/common/http-exception.filter.ts`:

```typescript
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
  Logger,
} from "@nestjs/common";
import type { Request, Response } from "express";

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const body = exception.getResponse();
      response.status(status).json(body);
      return;
    }

    this.logger.error(
      `Unhandled error on ${request.method} ${request.url}`,
      exception instanceof Error ? exception.stack : String(exception),
    );
    response.status(HttpStatus.INTERNAL_SERVER_ERROR).json({
      statusCode: 500,
      message: "Internal server error",
    });
  }
}
```

Register in `main.ts`:

```typescript
import { AllExceptionsFilter } from "./common/http-exception.filter";

app.useGlobalFilters(new AllExceptionsFilter());
```

**Logic:** Without a filter, an unexpected error returns Nest's default body, which can leak stack traces. This filter logs the detail server-side and returns a generic message to the client — the correct split: **full detail in logs, minimal detail to the client.**

### Step 6.7 — Response shaping with an interceptor

`src/common/logging.interceptor.ts`:

```typescript
import { CallHandler, ExecutionContext, Injectable, NestInterceptor } from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    return next.handle().pipe(
      map((data) => ({ success: true, data, timestamp: new Date().toISOString() })),
    );
  }
}
```

Apply it to a controller or globally:

```typescript
app.useGlobalInterceptors(new TransformInterceptor());
```

**Logic:** Interceptors wrap the handler, so they run **before** (code above `next.handle()`) and **after** (the `map` below it) the handler. That makes them the right place for logging, timing, caching, and response shaping. Pipes are only for the input.

### Step 6.8 — Prisma 7 equivalents for this lesson

```typescript
// create with data wrapper
return this.prisma.user.create({ data: { title: dto.title } });
// find many / find first
return this.prisma.todo.findMany();
return this.prisma.todo.findUnique({ where: { id } });
// update / delete
return this.prisma.todo.update({ where: { id }, data: dto });
return this.prisma.todo.delete({ where: { id } });
// delete many
return this.prisma.todo.deleteMany();
// transaction
return this.prisma.$transaction(async (tx) => { /* ... */ });
```

Validation, pipes, filters and interceptors are **identical** in Prisma 7 — they are all NestJS features.

**Exercise:**
1. Add `@MaxLength(500)` to a `notes` field on `CreateTodoDto`.
2. Add a global `HttpExceptionFilter` that maps `sqlState === "23505"` to `ConflictException`.
3. Add an interceptor that logs `method + url + duration in ms` and returns `{ success, data }`.

---

## Lesson 7: Auth with JWT (Register, Login, Guards)

**Time:** ~90 minutes
**Goal:** Your own email/password auth. Passwords hashed, tokens signed, routes protected.

### Step 7.1 — Understand the flow first

```text
1. POST /auth/register  -> hash password, insert user, return 201 (no token)
2. POST /auth/login     -> verify password, sign a JWT, return { accessToken }
3. Client sends         -> Authorization: Bearer <accessToken>
4. JwtAuthGuard         -> verify signature + expiry, attach payload to request
5. Controller           -> read req.user
```

**Logic:** A **JWT** is a signed JSON blob: `header.payload.signature`. The payload contains claims (`sub`, `email`, `role`). The signature is created with a **secret only the server knows**, so the server can prove the token was not forged. Because the payload is only encoded (not encrypted), never put secrets in it.

### Step 7.2 — Install packages

**CMD**
```cmd
npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcrypt
npm install -D @types/passport-jwt @types/bcrypt
```

**PowerShell**
```powershell
npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcrypt
npm install -D @types/passport-jwt @types/bcrypt
```

| Package | Role |
|---|---|
| `@nestjs/jwt` | signs and verifies tokens |
| `@nestjs/passport` + `passport-jwt` | the JWT strategy and guard |
| `passport` | the strategy framework |
| `bcrypt` | password hashing (never store plain passwords) |

### Step 7.3 — Add the User model

Add to `src/prisma/contract.prisma`:

```prisma
// use prisma-8

enum Role {
  USER
  ADMIN
}

model User {
  id           String   @id @default(cuid(2))
  email        String   @unique
  passwordHash String
  name         String?
  role         Role     @default(USER)
  createdAt    DateTime @default(now())
  todos        Todo[]
}

// ... keep your Todo model, and add userId:
model Todo {
  id        String   @id @default(cuid(2))
  title     String
  completed Boolean  @default(false)
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

Then run the loop:

**CMD**
```cmd
npx prisma contract emit
npx prisma migration plan
npx prisma db migrate
```

**PowerShell**
```powershell
npx prisma contract emit
npx prisma migration plan
npx prisma db migrate
```

**Logic:**
- `passwordHash String` — we store the **hash**, never the password. Column named explicitly so nobody confuses it with a password.
- `role Role @default(USER)` — an enum. Prisma 8 stores a plain enum as **text** (not a PostgreSQL `CREATE TYPE`). Simpler for learning.
- `userId String` + `user User @relation(...)` — the **foreign key**. `Todo` belongs to one `User`; `User.todos Todo[]` is the other side. `references: [id]` says it points at `User.id`.

> Making `userId` required means existing todos without an owner break. If that happens, make it optional for now: `userId String?` and `user User? @relation(...)`.

### Step 7.4 — Environment variables

`.env`:

```env
DATABASE_URL="postgresql://nestapp:nestpass@localhost:5432/nestdb"
JWT_SECRET="dev-only-change-me-to-a-long-random-string"
JWT_EXPIRES_IN="1h"
```

Generate a real secret:

**CMD**
```cmd
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

**PowerShell**
```powershell
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

**Logic:** The secret must be long and random and **different in production**. In production it comes from the hosting provider's environment variables, not from a committed `.env`.

### Step 7.5 — Generate the auth module

**CMD**
```cmd
nest g module auth
nest g service auth
nest g controller auth
nest g guard jwt-auth
```

**PowerShell**
```powershell
nest g module auth
nest g service auth
nest g controller auth
nest g guard jwt-auth
```

### Step 7.6 — DTOs

`src/auth/dto/register.dto.ts`:

```typescript
import { IsEmail, IsOptional, IsString, MinLength } from "class-validator";

export class RegisterDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsOptional()
  @IsString()
  name?: string;
}
```

`src/auth/dto/login.dto.ts`:

```typescript
import { IsEmail, IsString, MinLength } from "class-validator";

export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

**Logic:** `@MinLength(8)` on login too — it rejects obviously wrong input early without a database lookup. (Do not compare error messages between "no such email" and "wrong password"; that leaks which emails exist. Return the same message for both.)

### Step 7.7 — Auth service

`src/auth/auth.service.ts`:

```typescript
import { ConflictException, Injectable, UnauthorizedException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import * as bcrypt from "bcrypt";
import { PrismaService } from "../prisma/prisma.service";
import { LoginDto } from "./dto/login.dto";
import { RegisterDto } from "./dto/register.dto";

@Injectable()
export class AuthService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly jwt: JwtService,
  ) {}

  async register(dto: RegisterDto) {
    const existing = await this.prisma.orm.public.User
      .where({ email: dto.email })
      .first();
    if (existing) {
      throw new ConflictException("Email already registered");
    }

    const passwordHash = await bcrypt.hash(dto.password, 12);

    const user = await this.prisma.orm.public.User.create({
      email: dto.email,
      passwordHash,
      name: dto.name,
    });

    return this.toPublicUser(user);
  }

  async login(dto: LoginDto) {
    const user = await this.prisma.orm.public.User
      .where({ email: dto.email })
      .first();

    // always run a comparison so timing does not reveal whether the email exists
    const hash = user?.passwordHash ?? "$2b$12$invalidinvalidinvalidinvalidinvalidinvalidinvalidinvalidinv";
    const ok = await bcrypt.compare(dto.password, hash);

    if (!user || !ok) {
      throw new UnauthorizedException("Invalid email or password");
    }

    const payload = { sub: user.id, email: user.email, role: user.role };

    return {
      accessToken: await this.jwt.signAsync(payload),
      user: this.toPublicUser(user),
    };
  }

  private toPublicUser(user: { id: string; email: string; name: string | null; role: string }) {
    return { id: user.id, email: user.email, name: user.name, role: user.role };
  }
}
```

**Logic, carefully:**
- `bcrypt.hash(password, 12)` — **12 rounds** of salting. Slow on purpose: it makes offline cracking expensive. Never store or log the plain password.
- `bcrypt.compare(plain, hash)` — returns `true`/`false`; it does not need the salt because the salt is inside the hash string.
- The dummy-hash trick on line 3 of `login` keeps the response time similar whether the email exists or not.
- `jwt.signAsync(payload)` signs with `JWT_SECRET`. We put `sub` (the standard claim for "subject" = user id), `email`, `role`.
- `toPublicUser` **strips `passwordHash`**. Never send it to a client — not in login, not anywhere.

### Step 7.8 — JWT module registration

`src/auth/auth.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { AuthController } from "./auth.controller";
import { AuthService } from "./auth.service";
import { JwtStrategy } from "./jwt.strategy";

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: "jwt" }),
    JwtModule.register({
      global: true,
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: process.env.JWT_EXPIRES_IN ?? "1h" },
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

**Logic:** `JwtModule.register({ global: true })` makes `JwtService` injectable anywhere without re-importing. `defaultStrategy: "jwt"` lets you write `@UseGuards(AuthGuard)` without naming the strategy. `process.env.JWT_EXPIRES_IN` is a **string**, but `expiresIn` expects a number of seconds or a string like `"1h"` — the string form works.

### Step 7.9 — JWT strategy (verifies the token)

`src/auth/jwt.strategy.ts`:

```typescript
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { PrismaService } from "../prisma/prisma.service";

export interface JwtPayload {
  sub: string;
  email: string;
  role: string;
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private readonly prisma: PrismaService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET!,
    });
  }

  async validate(payload: JwtPayload) {
    const user = await this.prisma.orm.public.User
      .where({ id: payload.sub })
      .first();

    if (!user) {
      throw new UnauthorizedException("User no longer exists");
    }

    return { id: user.id, email: user.email, role: user.role };
  }
}
```

**Logic:**
- `ExtractJwt.fromAuthHeaderAsBearerToken()` — pulls the token from `Authorization: Bearer xxx`.
- `ignoreExpiration: false` — reject expired tokens. This is the single most important line for security.
- `validate()` runs after the signature and expiry check pass. It is your chance to reject a valid-but-stale token (deleted user, banned user). Whatever it returns becomes `request.user`.
- We return only safe fields, never the whole database row.

### Step 7.10 — The guard

`src/auth/jwt-auth.guard.ts`:

```typescript
import { ExecutionContext, Injectable } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { AuthGuard } from "@nestjs/passport";
import { IS_PUBLIC_KEY } from "./public.decorator";

@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true; // skip authentication entirely
    }

    return super.canActivate(context);
  }
}
```

`src/auth/public.decorator.ts`:

```typescript
import { SetMetadata } from "@nestjs/common";

export const IS_PUBLIC_KEY = "isPublic";

export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

**Logic:** By default **everything is protected** (deny by default is safer than allow by default). Opt out per route with `@Public()`. The reflector reads metadata set by `@SetMetadata`, checking the handler first, then the class — so `@Public()` on the method wins over a class-level one.

### Step 7.11 — Register global modules and make auth global

`src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { APP_GUARD } from "@nestjs/core";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { AuthModule } from "./auth/auth.module";
import { JwtAuthGuard } from "./auth/jwt-auth.guard";
import { PrismaModule } from "./prisma/prisma.module";
import { TodosModule } from "./todos/todos.module";

@Module({
  imports: [PrismaModule, AuthModule, TodosModule],
  controllers: [AppController],
  providers: [
    AppService,
    { provide: APP_GUARD, useClass: JwtAuthGuard }, // every route is protected by default
  ],
})
export class AppModule {}
```

Mark the two auth routes and health route public — in `auth.controller.ts`:

```typescript
@Public()
@Post("register")
register(@Body() dto: RegisterDto) {
  return this.authService.register(dto);
}

@Public()
@Post("login")
login(@Body() dto: LoginDto) {
  return this.authService.login(dto);
}
```

And in `app.controller.ts`:

```typescript
@Public()
@Get()
getHello() {
  return { message: "Hello World!" };
}
```

### Step 7.12 — Current user decorator

`src/auth/current-user.decorator.ts`:

```typescript
import { createParamDecorator, ExecutionContext } from "@nestjs/common";
import type { AuthUser } from "../auth/user.types";

export const CurrentUser = createParamDecorator(
  (data: keyof AuthUser | undefined, ctx: ExecutionContext) => {
    const user = ctx.switchToHttp().getRequest().user as AuthUser;
    return data ? user?.[data] : user;
  },
);
```

Usage in the controller:

```typescript
@Get("me")
me(@CurrentUser() user: AuthUser) {
  return user;
}

@Get("mine")
mine(@CurrentUser("id") userId: string) {
  return this.todosService.findMine(userId);
}
```

**Logic:** A custom param decorator reads from the request and hands you one value. Without the key it gives the whole user object; with `"id"` it gives just the id. This is how you avoid `req.user.id` scattered everywhere.

### Step 7.13 — Controller

`src/auth/auth.controller.ts`:

```typescript
import { Body, Controller, Get, Post, UseGuards } from "@nestjs/common";
import { AuthService } from "./auth.service";
import { CurrentUser } from "./current-user.decorator";
import { LoginDto } from "./dto/login.dto";
import { RegisterDto } from "./dto/register.dto";
import { Public } from "./public.decorator";
import type { AuthUser } from "./user.types";

@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Public()
  @Post("register")
  register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }

  @Public()
  @Post("login")
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }

  @Get("me")
  me(@CurrentUser() user: AuthUser) {
    return this.authService.me(user.id);
  }
}
```

Add to `AuthService`:

```typescript
  async me(id: string) {
    const user = await this.prisma.orm.public.User.where({ id }).first();
    if (!user) throw new NotFoundException("User not found");
    return this.toPublicUser(user);
  }
```

### Step 7.14 — Test the whole flow

**CMD**
```cmd
curl -X POST http://localhost:3000/auth/register -H "Content-Type: application/json" -d "{\"email\":\"a@b.com\",\"password\":\"password123\",\"name\":\"Ali\"}"
curl -X POST http://localhost:3000/auth/login -H "Content-Type: application/json" -d "{\"email\":\"a@b.com\",\"password\":\"password123\"}"
curl http://localhost:3000/todos
curl http://localhost:3000/todos -H "Authorization: Bearer PASTE_TOKEN"
curl http://localhost:3000/auth/me -H "Authorization: Bearer PASTE_TOKEN"
```

**PowerShell**
```powershell
Invoke-RestMethod -Uri http://localhost:3000/auth/register -Method POST -ContentType "application/json" -Body '{"email":"a@b.com","password":"password123","name":"Ali"}'
$login = Invoke-RestMethod -Uri http://localhost:3000/auth/login -Method POST -ContentType "application/json" -Body '{"email":"a@b.com","password":"password123"}'
$token = $login.accessToken
Invoke-RestMethod -Uri http://localhost:3000/todos
Invoke-RestMethod -Uri http://localhost:3000/todos -Headers @{ Authorization = "Bearer $token" }
Invoke-RestMethod -Uri http://localhost:3000/auth/me -Headers @{ Authorization = "Bearer $token" }
```

`requests.http` version with a saved token:

```http
# @name login
POST http://localhost:3000/auth/login
Content-Type: application/json

{ "email": "a@b.com", "password": "password123" }

### protected, no token -> 401
GET http://localhost:3000/todos

### protected with token
# @token = {{login.response.body.$.accessToken}}
GET http://localhost:3000/todos
Authorization: Bearer {{token}}

### who am I
GET http://localhost:3000/auth/me
Authorization: Bearer {{token}}

### logout is client-side: just discard the token
```

Check the results:
- `/todos` with no token → **401 Unauthorized** (the guard works)
- `/todos` with a valid token → 200
- Login with a wrong password → **401** with "Invalid email or password"
- Register with the same email twice → **409**

**Logic:** A 401 without a token and 401 with a bad token are both correct. If you get 200 without a token, the `APP_GUARD` is not registered or the route has `@Public()`.

### Step 7.15 — Scope todos to the logged-in user

`src/todos/todos.service.ts`:

```typescript
findMine(userId: string) {
  return this.prisma.orm.public.Todo
    .where({ userId })
    .orderBy("createdAt")
    .all();
}

async createFor(userId: string, dto: CreateTodoDto) {
  return this.prisma.orm.public.Todo.create({
    title: dto.title,
    userId,
  });
}

async removeOwn(id: string, userId: string) {
  const todo = await this.prisma.orm.public.Todo.where({ id }).first();
  if (!todo) throw new NotFoundException("Todo not found");
  if (todo.userId !== userId) {
    throw new ForbiddenException("This is not your todo");
  }
  await this.prisma.orm.public.Todo.where({ id }).delete();
  return { message: "deleted" };
}
```

**Logic:** **Always scope queries by the user's id.** Checking ownership in the controller *after* fetching is fine, but filtering in the query (`where({ id, userId })`) is better: it never loads someone else's row. Both are shown so you see the difference.

### Step 7.16 — Role-based guard (RBAC)

`src/auth/roles.decorator.ts`:

```typescript
import { SetMetadata } from "@nestjs/common";

export const ROLES_KEY = "roles";
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

`src/auth/roles.guard.ts`:

```typescript
import { CanActivate, ExecutionContext, Injectable } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { ROLES_KEY } from "./roles.decorator";
import type { AuthUser } from "./user.types";

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext) {
    const required = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!required || required.length === 0) {
      return true; // no roles required
    }

    const user = context.switchToHttp().getRequest().user as AuthUser | undefined;

    if (!user || !required.includes(user.role)) {
      return false; // -> 403 Forbidden
    }

    return true;
  }
}
```

Use it:

```typescript
@Roles("ADMIN")
@UseGuards(RolesGuard)
@Get("admin/users")
adminUsers() {
  return this.authService.listAll();
}
```

**Logic:** `@UseGuards(RolesGuard)` on that one route. Guards can be applied per route, per controller, or globally via `APP_GUARD`. Roles are coarse — for real permissions you often need ownership checks per row (Lesson 9).

### Step 7.17 — Auth cheat sheet

| Concept | One-liner |
|---|---|
| hash | one-way; you can check but never recover |
| salt | random per password, built into the hash |
| JWT | signed, readable, expiring token |
| `sub` claim | standard claim for the user id |
| guard | decides *whether* a request proceeds |
| strategy | how a guard verifies the credential |
| `expiresIn` | how long a token is valid |
| refresh token | long-lived, used to get a new access token |
| 401 vs 403 | 401 = "who are you?", 403 = "I know you, but no" |

**Security checklist:**
- passwords hashed with bcrypt (12+ rounds)
- `passwordHash` never returned by any endpoint
- `ignoreExpiration: false`
- guards applied globally (deny by default)
- queries scoped by `userId`
- `JWT_SECRET` long, random, different per environment
- `.env` in `.gitignore`

**Exercise:**
1. Add `GET /auth/me/todos` returning the logged-in user's todos.
2. Write a `@Roles("ADMIN")` route that lists all users, and add an `ADMIN` user by hand in psql:
   ```sql
   UPDATE "User" SET role = 'ADMIN' WHERE email = 'a@b.com';
   ```
3. Prove that user A cannot delete user B's todo (expect 403).

---

## Lesson 8: Auth0 (Protect an API with Auth0-issued tokens)

**Time:** ~60 minutes
**Goal:** Stop issuing tokens yourself. Auth0 becomes the identity provider; your API becomes a **resource server** that validates Auth0's access tokens.

### Step 8.1 — What changes, and what does not

Lesson 7 (local auth) vs Lesson 8 (Auth0):

| Part | Lesson 7 (local) | Lesson 8 (Auth0) |
|---|---|---|
| Who issues the token | your API | **Auth0** |
| Token signing key | your `JWT_SECRET` (HS256) | Auth0's private key (**RS256**, asymmetric) |
| Register / Login routes | you build them | **Auth0's Universal Login** page — you build nothing |
| Your API's job | sign + verify | **verify only** |
| Passwords | you hash them | Auth0 stores them |
| Where the key comes from | your `.env` | Auth0's **JWKS** URL, fetched over HTTPS |
| Endpoints still protected | yes | yes, same guard concept |

**Logic — the key idea (RS256 / asymmetric):**
With a shared secret, the same key signs *and* verifies. With RS256, Auth0 holds a **private** key to sign, and publishes a **public** key at a JWKS URL. Your API downloads the public key and can verify but cannot forge. That is why you cannot just hardcode Auth0's token — you must fetch their public keys, and they rotate over time.

### Step 8.2 — What "resource server" means

An Auth0 **API** (resource server) is a logical API registered in Auth0 with an **Audience** (a URI like `https://api.example.com`). You ask Auth0 for a token *for that audience*, and your API only accepts tokens whose `aud` matches.

```text
Angular/React/mobile client
   -> Auth0 Universal Login (login + consent)
   -> gets access_token (aud = https://api.example.com, iss = https://tenant.auth0.com/)
   -> sends it to YOUR Nest API as Bearer token
   -> your Nest API validates signature (JWKS) + iss + aud + exp
   -> returns data
```

Your Nest API never sees the password. That is the whole point of moving to Auth0.

### Step 8.3 — Create the Auth0 tenant and API

1. Sign up at https://auth0.com (a free "Free" plan is enough for learning).
2. Create a **Tenant** (use the free dev tenant, e.g. `dev-xxxxx.us.auth0.com`).
3. In the sidebar: **Applications → Applications → Create Application**
   - Type: **Single Page Web Application**
   - Name: `nest-learning`
   - Settings → Allowed Callback URLs: `http://localhost:5173` (or any placeholder for now)
   - Allowed Web Origins: `http://localhost:5173`
4. In the sidebar: **Applications → APIs → Create API**
   - Name: `Nest Learning API`
   - Identifier (**Audience**): `https://api.nest-learning.com` (must be a URI)
   - Signing Algorithm: **RS256** (default; do not change)
5. Open the API → **Settings** → note:
   - **Audience** = `https://api.nest-learning.com`
   - **Issuer** = `https://dev-xxxxx.us.auth0.com/`
   - **JWKS URI** = `https://dev-xxxxx.us.auth0.com/.well-known/jwks.json`
6. Optional: **Permissions** tab → create `read:todos` and `write:todos` and add them as **RBAC** permissions. Enable "Enable RBAC" in Settings.

Write these three values down. They go in your `.env` next.

### Step 8.4 — Get a token (the "get token" trick, no frontend needed)

Auth0's **Getting Started → API → Access Token** page has a "Try It" widget that shows:

- **Authorize** (login page URL)
- **Audience**
- a **curl** command with a client id / client secret / grant type

Simpler for a first request — **Client Credentials Grant** (machine-to-machine). Use your API's client:

1. **Applications → Applications → your SPA app → Settings → Show Advanced Settings → Issuer Based Holder of Key Proof** (leave off) — instead create a new **Machine to Machine Application**:
   - Applications → Create Application → type: **Machine to Machine Application**, name `nest-cli`
2. Open it → **Settings** → copy **Client ID** and **Client Secret**; set **Authorize** to allow your API's `read:todos` permission.
3. Get a token:

**CMD**
```cmd
curl -X POST "https://dev-xxxxx.us.auth0.com/oauth/token" -H "Content-Type: application/json" -d "{\"client_id\":\"YOUR_CLIENT_ID\",\"client_secret\":\"YOUR_CLIENT_SECRET\",\"audience\":\"https://api.nest-learning.com\",\"grant_type\":\"client_credentials\"}"
```

**PowerShell**
```powershell
$body = @{
  client_id     = 'YOUR_CLIENT_ID'
  client_secret = 'YOUR_CLIENT_SECRET'
  audience      = 'https://api.nest-learning.com'
  grant_type    = 'client_credentials'
} | ConvertTo-Json
Invoke-RestMethod -Uri "https://dev-xxxxx.us.auth0.com/oauth/token" -Method POST -ContentType "application/json" -Body $body
```

You get:

```json
{ "access_token": "eyJhbGciOiJSUzI1NiIs...", "expires_in": 86400, "token_type": "Bearer" }
```

**Logic:** the client-credentials grant has **no user**. It represents your application acting for itself, so there is no `sub` user and no `scope` from a user. It is perfect for testing a protected API from curl. For testing with a *user's* identity, use Auth0's **Try It** widget with the Authorization code + PKCE flow.

### Step 8.5 — Install the packages

**CMD**
```cmd
npm install passport-jwt jwks-rsa
```

**PowerShell**
```powershell
npm install passport-jwt jwks-rsa
```

| Package | Role |
|---|---|
| `passport-jwt` | the strategy that reads the Bearer token |
| `jwks-rsa` | downloads Auth0's **public** signing keys (JWKS) and caches them |

`passport`, `@nestjs/passport`, `@types/passport-jwt` are already installed from Lesson 7.

### Step 8.6 — Environment variables

`.env`:

```env
DATABASE_URL="postgresql://nestapp:nestpass@localhost:5432/nestdb"

AUTH0_DOMAIN="dev-xxxxx.us.auth0.com"
AUTH0_AUDIENCE="https://api.nest-learning.com"
AUTH0_CLIENT_ID="YOUR_CLIENT_ID"
AUTH0_CLIENT_SECRET="YOUR_CLIENT_SECRET"
```

Derive the JWKS URL from the domain:

```text
https://{AUTH0_DOMAIN}/.well-known/jwks.json
```

**Logic:** the issuer is `https://{AUTH0_DOMAIN}/` (with a trailing slash). The `audience` must exactly match what you registered as the API identifier. Keep the client secret **only** for scripts; the API itself never needs it.

### Step 8.7 — The Auth0 JWT strategy

`src/auth/auth0.strategy.ts`:

```typescript
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { createRemoteJWKSet } from "jwks-rsa";

const DOMAIN = process.env.AUTH0_DOMAIN!;
const AUDIENCE = process.env.AUTH0_AUDIENCE!;

@Injectable()
export class Auth0Strategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      // 1. where the token is in the request
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      // 2. how to get the public keys that verify the signature
      secretOrKeyProvider: createRemoteJWKSet(
        new URL(`https://${DOMAIN}/.well-known/jwks.json`),
        { cache: true, rateLimit: true, jwksRequestsPerMinute: 5 },
      ),
      // 3. who is allowed to call me
      audience: AUDIENCE,
      issuer: `https://${DOMAIN}/`,
      // 4. algorithm Auth0 uses for RS256
      algorithms: ["RS256"],
    });
  }

  validate(payload: { sub?: string; scope?: string; permissions?: string[] }) {
    if (!payload.sub) {
      throw new UnauthorizedException("Token has no subject");
    }
    return {
      id: payload.sub,
      scope: payload.scope?.split(" ") ?? [],
      permissions: payload.permissions ?? [],
    };
  }
}
```

**Logic, every option:**
- `jwtFromRequest` — same as before: `Authorization: Bearer ...`.
- `secretOrKeyProvider: createRemoteJWKSet(...)` — this is the RS256 piece. It fetches Auth0's **public** keys over HTTPS and caches them. `rateLimit: true` protects Auth0's endpoint from being hammered; `cache: true` means only the first request pays the network cost.
- `audience: AUDIENCE` — the token must be **for your API**. A token minted for a different API is rejected, even if signed by Auth0.
- `issuer: 'https://{DOMAIN}/'` — must exactly match Auth0's issuer, so a token from someone else's tenant fails.
- `algorithms: ['RS256']` — whitelist the algorithm. This blocks the classic `alg: none` and HMAC-confusion attacks, where a forged token says it is signed with HS256 using your own public key as the "secret".
- `validate()` — only runs after signature, expiry, audience and issuer all pass. Put whatever you need on `request.user`.

### Step 8.8 — Swap the guard to Auth0

`src/auth/jwt-auth.guard.ts` — change the base class from `AuthGuard("jwt")` to `AuthGuard("auth0")`:

```typescript
import { Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Injectable()
export class Auth0Guard extends AuthGuard("auth0") {}
```

And in `src/app.module.ts`, replace the guard provider:

```typescript
providers: [
  AppService,
  { provide: APP_GUARD, useClass: Auth0Guard }, // Auth0 instead of local JWT
]
```

Mark the routes you want open with `@Public()` — `@Public()`, `@Post('login')` etc. — and delete your `AuthController.register/login` if you fully switch to Auth0. Keep `@Roles` guard from Lesson 7 and repurpose it to check Auth0 permissions.

**Logic:** the `@Public()` decorator, the `APP_GUARD` pattern, the reflector, and the roles guard are all **NestJS** concepts — they do not change. Only the *strategy* changed: instead of "verify with my secret", it is "verify with Auth0's public key".

### Step 8.9 — Protect routes with Auth0 permissions

`src/auth/permissions.guard.ts`:

```typescript
import { CanActivate, ExecutionContext, ForbiddenException, Injectable } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { ROLES_KEY } from "./roles.decorator";

@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext) {
    const required =
      this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
        context.getHandler(),
        context.getClass(),
      ]) ?? [];

    if (required.length === 0) return true;

    const user = context.switchToHttp().getRequest().user as
      | { permissions?: string[] }
      | undefined;

    const granted = user?.permissions ?? [];

    if (!required.every((p) => granted.includes(p))) {
      throw new ForbiddenException(
        `Missing permission(s): ${required.filter((p) => !granted.includes(p)).join(", ")}`,
      );
    }

    return true;
  }
}
```

Usage:

```typescript
@Permissions("read:todos")   // your own decorator, same SetMetadata pattern
@UseGuards(PermissionsGuard)
@Get()
findAll() {
  return this.todosService.findAll();
}
```

**Logic:** Auth0 permissions are **space-separated scopes** in the `scope` claim. A client-credentials token without RBAC enabled has an **empty** `scope` and **no** `permissions` claim, so the guard throws 403. To get permissions on a client-credentials token, enable RBAC + "Add Permissions in the Access Token" in the API settings, and authorize the permission on the M2M app. For user tokens from the Authorization Code flow, permissions come from the user's assigned roles.

### Step 8.10 — Test it

**CMD**
```cmd
curl http://localhost:3000/todos
curl http://localhost:3000/todos -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIs..."
curl http://localhost:3000/todos -H "Authorization: Bearer garbage"
```

**PowerShell**
```powershell
Invoke-RestMethod -Uri http://localhost:3000/todos
$token = "eyJhbGciOiJSUzI1NiIs..."
Invoke-RestMethod -Uri http://localhost:3000/todos -Headers @{ Authorization = "Bearer $token" }
Invoke-RestMethod -Uri http://localhost:3000/todos -Headers @{ Authorization = "Bearer garbage" }
```

Expected:
- no token → **401**
- garbage token → **401** (signature fails)
- valid token → **200**
- valid token but wrong audience (mint one for a different API) → **401** (audience mismatch)
- expired token → **401**

**Logic:** a 401 from `garbage` proves the signature check is real. If garbage were accepted, your API would be an open door.

### Step 8.11 — Common Auth0 failures and fixes

| Symptom | Cause | Fix |
|---|---|---|
| `secret or public key must be provided` | `secretOrKeyProvider` not set | you used `secretOrKey` (Lesson 7 style) with RS256 |
| `jwt audience invalid` | `audience` mismatch or missing | match the API identifier exactly, no trailing slash |
| `jwt issuer invalid` | issuer mismatch | `https://{DOMAIN}/` with trailing slash |
| `unable to download JWKS` / `request to jwks endpoint failed` | `AUTH0_DOMAIN` wrong, or offline | check the domain, check your internet |
| `permission denied` | no permissions on the token | enable RBAC in the API, authorize the M2M app |
| 403 from the permissions guard | scope is empty | mint a new token after enabling RBAC |
| tokens work in curl, fail in a browser app | CORS | add the origin to Auth0's **Allowed Web Origins** |

### Step 8.12 — Choosing between Lesson 7 and Lesson 8

| | Local (Lesson 7) | Auth0 (Lesson 8) |
|---|---|---|
| Learn | how JWT + guards + hashing work from scratch | how to integrate a real IdP |
| Good for | learning, small self-contained apps | production, multiple clients, social login, MFA |
| Code you own | register, login, hashing, JWKS-less validation | only validation |
| Effort to add social login | high | zero (Auth0 has 50+ social providers) |

**Logic:** both use the same Nest building blocks: a strategy, a guard, `@Public()`, and `APP_GUARD`. Once you have Lesson 7, Lesson 8 is a ~40-line change. That is the real lesson.

**Exercise:**
1. Create two Auth0 APIs with different audiences; prove a token for A is rejected by B.
2. Create a permission `delete:anything` in Auth0, require it on `DELETE /todos/:id`, and confirm 403 without it.
3. Compare: log the `sub` claim from an Auth0 token vs your local JWT's `sub` — note the difference (`auth0|abc123` vs a cuid).

---

## Lesson 9: Relations, Pagination, Filtering, Config

**Time:** ~60 minutes
**Goal:** Join data, paginate, filter, load env config properly, and add Swagger.

### Step 9.1 — Relations in the contract

You already have a one-to-many relation (`User` -> `Todo[]`). The two sides:

```prisma
model User {
  id    String @id @default(cuid(2))
  todos Todo[]          // <- "one user has many todos"
}

model Todo {
  id     String @id @default(cuid(2))
  userId String
  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

| Prisma | SQL |
|---|---|
| `userId String` | `userId` column, `NOT NULL` |
| `@relation(fields: [userId], references: [id])` | foreign key from `Todo.userId` to `User.id` |
| `todos Todo[]` | no column; the reverse lookup |
| `@unique` | `UNIQUE` constraint (so `findUnique` works on it) |
| `@index` | `CREATE INDEX` |
| `onDelete: Cascade` | deleting the user deletes their todos |

Read a relation:

```typescript
// one user with their todos
const user = await db.orm.public.User
  .where({ id: userId })
  .include("todos")
  .first();

// many-to-many needs a join table model
model Post {
  id        String @id @default(cuid(2))
  tags      Tag[]
}

model Tag {
  id    String @id @default(cuid(2))
  posts Post[]
}

model PostTag {              // the join model
  postId String
  tagId  String
  post   Post   @relation(fields: [postId], references: [id])
  tag    Tag    @relation(fields: [tagId], references: [id])

  @@id([postId, tagId])     // composite primary key
}
```

Then `.include("tags")` works on `Post`.

**Logic:** a foreign key is a column holding another table's primary key. `include` tells Prisma to fetch the related rows in the **same** query (a join), not with a second query — so you get the user and their todos in one round trip. The join table for many-to-many has no surrogate key, so it uses a **composite** primary key `@@id([a, b])`.

### Step 9.2 — Pagination

`src/todos/dto/pagination.dto.ts`:

```typescript
import { Type } from "class-transformer";
import { IsInt, IsOptional, Max, Min } from "class-validator";

export class PaginationDto {
  @IsOptional()
  @Type(() => Number)      // "2" -> 2, because transform: true is on
  @IsInt()
  @Min(1)
  page = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit = 10;

  get skip() {
    return (this.page - 1) * this.limit;
  }
}
```

Service:

```typescript
findPaginated(query: PaginationDto) {
  return this.prisma.orm.public.Todo
    .where({})
    .orderBy("createdAt", "desc")
    .limit(query.limit)
    .offset(query.skip)
    .all();
}
```

Controller:

```typescript
@Get()
findAll(@Query() query: PaginationDto) {
  return this.todosService.findPaginated(query);
}
```

Test:

**CMD**
```cmd
curl "http://localhost:3000/todos?page=2&limit=5"
curl "http://localhost:3000/todos?page=0"
curl "http://localhost:3000/todos?limit=999"
```

**PowerShell**
```powershell
Invoke-RestMethod -Uri "http://localhost:3000/todos?page=2&limit=5"
Invoke-RestMethod -Uri "http://localhost:3000/todos?page=0"
Invoke-RestMethod -Uri "http://localhost:3000/todos?limit=999"
```

**Logic:** `page=0` → **400** from `@Min(1)`. `limit=999` → **400** from `@Max(100)` (a deliberate cap: without it one request can ask for a million rows). `skip = (page-1) * limit` converts page numbers to SQL `OFFSET`. `@Type(() => Number)` is required because query params arrive as strings.

`orderBy("createdAt", "desc")` matters for correctness: without a stable sort, rows can repeat or vanish between pages.

### Step 9.3 — Filtering and search

`src/todos/dto/filter-todo.dto.ts`:

```typescript
import { Transform } from "class-transformer";
import { IsBoolean, IsOptional, IsString } from "class-validator";

export class FilterTodoDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @Transform(({ value }) => value === "true" || value === true)
  @IsBoolean()
  completed?: boolean;
}
```

Service:

```typescript
findFiltered(filter: FilterTodoDto, page: PaginationDto) {
  let query = this.prisma.orm.public.Todo;

  if (filter.completed !== undefined) {
    query = query.where({ completed: filter.completed });
  }
  if (filter.search) {
    query = query.where((p) => p.title.ilike(`%${filter.search}%`));
  }

  return query
    .orderBy("createdAt", "desc")
    .limit(page.limit)
    .offset(page.skip)
    .all();
}
```

**Logic:**
- `where((p) => p.title.ilike(...))` is the **callback filter form** for conditions an object cannot express. `ilike` = SQL `ILIKE`, case-insensitive. `%` is a wildcard in SQL `LIKE`. So `%nest%` means "contains nest".
- `filter.completed !== undefined` matters: `where({ completed: false })` finds only `false`, while skipping the filter finds both. Checking `undefined` is different from checking falsy.
- **`ilike` with user input is SQL-injection-safe** because Prisma parameterizes the query — the `%` is part of the value, not part of the SQL. Never build SQL by string concatenation.

### Step 9.4 — Configuration with `@nestjs/config`

**CMD**
```cmd
npm install @nestjs/config
```

```powershell
npm install @nestjs/config
```

`src/config/env.validation.ts`:

```typescript
export function validateEnv(config: Record<string, unknown>) {
  const required = ["DATABASE_URL", "JWT_SECRET", "JWT_EXPIRES_IN"];
  for (const key of required) {
    if (!config[key]) {
      throw new Error(`Missing required env var: ${key}`);
    }
  }
  return config;
}
```

`src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { APP_GUARD } from "@nestjs/core";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { AuthModule } from "./auth/auth.module";
import { Auth0Guard } from "./auth/auth0.guard";
import { validateEnv } from "./config/env.validation";
import { PrismaModule } from "./prisma/prisma.module";
import { TodosModule } from "./todos/todos.module";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,            // available everywhere
      validate: validateEnv,     // fail fast on missing vars
    }),
    PrismaModule,
    AuthModule,
    TodosModule,
  ],
  controllers: [AppController],
  providers: [
    AppService,
    { provide: APP_GUARD, useClass: Auth0Guard },
  ],
})
export class AppModule {}
```

Read a variable anywhere:

```typescript
import { ConfigService } from "@nestjs/config";

@Injectable()
export class AuthService {
  constructor(private readonly config: ConfigService) {}

  private get jwtSecret() {
    return this.config.getOrThrow<string>("JWT_SECRET");
  }
}
```

**Logic:** `ConfigModule.forRoot({ isGlobal: true })` loads `.env` into `process.env` once, app-wide. `getOrThrow` fails loudly if the variable is missing instead of silently passing `undefined` into `jwt.sign` (which would produce tokens signed with the literal string "undefined" — a nasty bug). `validate` fails at **boot**, not at the first request.

Add `.env` to `.gitignore`:

```gitignore
.env
```

Commit `.env.example` with empty values instead.

### Step 9.5 — Swagger (interactive API docs)

```bash
npm install @nestjs/swagger
```

```powershell
npm install @nestjs/swagger
```

`src/main.ts`:

```typescript
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }),
  );

  const config = new DocumentBuilder()
    .setTitle("Nest Learning API")
    .setDescription("CRUD + Auth with Prisma 8 and PostgreSQL")
    .setVersion("1.0")
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup("docs", app, document);

  await app.listen(3000);
}
bootstrap();
```

Open `http://localhost:3000/docs`.

Annotate a controller so the docs show your schema:

```typescript
@ApiTags("todos")
@Controller("todos")
export class TodosController { ... }
```

```typescript
@ApiProperty({ example: "learn NestJS" })
title: string;

@ApiProperty({ required: false, example: false })
completed?: boolean;
```

**Logic:** Swagger generates `/docs` from your decorators, so the docs cannot drift from the code. `addBearerAuth()` adds the "Authorize" button so you can paste a JWT in the UI and call protected routes without curl. DTO classes double as the request/response schema, which is why DTOs are worth writing even for a small API.

### Step 9.6 — Global prefix and versioning

`src/main.ts`:

```typescript
app.setGlobalPrefix("api");
app.enableVersioning({ type: VersioningType.URI });
```

Routes become `/api/v1/todos` (add `@Version(VERSION_NEUTRAL)` for unversioned ones).

**Logic:** a global prefix keeps your domain clean (`/api/...`). URI versioning lets you ship `v2` beside `v1` while old clients keep working. Add it early; retrofitting it is painful.

### Step 9.7 — CORS (needed when a frontend calls the API)

```typescript
app.enableCors({
  origin: ['http://localhost:5173'],
  credentials: true,
});
```

**Logic:** browsers block cross-origin requests by default. In production, list your real front-end domain. `origin: true` (allow any) is fine for local curl work but wrong for production.

### Step 9.8 — Nest built-in exceptions reference

| Exception | Status | Use for |
|---|---|---|
| `BadRequestException` | 400 | invalid input that passed validation |
| `UnauthorizedException` | 401 | missing/invalid token, wrong password |
| `ForbiddenException` | 403 | valid token, not allowed |
| `NotFoundException` | 404 | resource does not exist |
| `ConflictException` | 409 | unique violation, duplicate |
| `UnprocessableEntityException` | 422 | syntactically valid, semantically wrong |
| `TooManyRequestsException` | 429 | rate limit |
| `InternalServerErrorException` | 500 | unexpected bug |
| `ServiceUnavailableException` | 503 | dependency down |

**Exercise:**
1. Add `GET /todos/stats` returning `{ total, completed, byUser: [{ userId, count }] }`. (For `byUser`, group by `userId` and `include` the user name.)
2. Add a `cursor` pagination mode next to page/limit: `GET /todos?after=<id>&limit=10` using `.orderBy('id').cursor()` and return `nextCursor`.
3. Add `@ApiTags`, `@ApiProperty`, and the `/docs` page. Try the "Authorize" button with a token from Lesson 7.
4. Add `@Version` support and confirm `/api/v1/todos` and `/api/v2/todos` both work.

---

## Lesson 10: Deployment (Simple)

**Time:** ~60 minutes
**Goal:** Put your API on the internet with a managed Postgres database.

### Step 10.1 — The plan

```text
Database: Neon   -> free managed Postgres, gives you a connection string
App:       Render -> free Node web service (or Railway, same idea)
```

Both have free tiers and both need only three things: **build command**, **start command**, **environment variables**.

### Step 10.2 — Create the Neon database

1. Sign up at https://neon.tech
2. **Create a project** → region near you → plan **Free**
3. Neon gives a connection string like:
   ```text
   postgresql://user:password@ep-xxx-pooler.region.aws.neon.tech/neondb?sslmode=require&channel_binding=prefer
   ```
4. Keep it safe. This string is a **production credential**.

**Logic:** Neon is Postgres, so your `contract.prisma` and all your Prisma code work unchanged. You do not rewrite anything for the cloud. Note the `?sslmode=require` — the hosted database requires TLS.

### Step 10.3 — Point your local app at Neon (test before deploying)

```env
DATABASE_URL="postgresql://user:password@ep-xxx-pooler.region.aws.neon.tech/neondb?sslmode=require"
```

```bash
npx prisma db init
npm run start:dev
```

Test the API. If it works locally against Neon, it will work on Render.

### Step 10.4 — Prepare the repo

```bash
git init
```

`.gitignore` (make sure these lines are present):

```gitignore
node_modules
dist
.env
.env.*
!.env.example
src/generated
*.log
```

Commit:

```bash
git add .
git commit -m "Nest + Prisma 8 + Postgres API"
```

Push to a **public** or **private** GitHub repo. Render can deploy from either.

**Logic:** never commit `.env`. Commit `.env.example` with placeholder values so others know which variables are needed.

### Step 10.5 — Deploy to Render

1. Push your code to GitHub.
2. Go to https://render.com → sign up → **New +** → **Web Service** → connect the repo.
3. Fill in:

| Field | Value |
|---|---|
| Root Directory | (blank, or `simple-api`) |
| Runtime | Node |
| Build Command | `npm ci && npm run build && npx prisma db migrate` |
| Start Command | `npm run start:prod` |
| Health Check Path | `/api/health` |

**4. Environment variables** (Render → Environment):

```
DATABASE_URL = postgresql://... (the Neon string)
JWT_SECRET   = (paste a fresh random secret)
JWT_EXPIRES_IN = 1h
AUTH0_DOMAIN = dev-xxxxx.us.auth0.com
AUTH0_AUDIENCE = https://api.nest-learning.com
NODE_ENV = production
PORT = (leave blank; Render injects it)
```

5. **Create Web Service**. Watch the deploy log.

**Logic — the build command is doing three things in order:**
1. `npm ci` — install **exactly** the versions in `package-lock.json` (reproducible).
2. `npm run build` — compile TypeScript to `dist/`. This also type-checks, so a type error fails the build (good).
3. `npx prisma db migrate` — apply migrations to the production database. Prisma 7 uses `npx prisma migrate deploy`.

If you used Prisma 8, `contract emit` runs as part of the Prisma commands. If you built with `prisma-client` (Prisma 7), make sure `generate` runs — add it to `package.json`:

```json
"scripts": {
  "build": "prisma generate && nest build",
  "start:prod": "node dist/main"
}
```

### Step 10.6 — Listen on the injected port

`src/main.ts`:

```typescript
const port = process.env.PORT ?? 3000;
await app.listen(port, '0.0.0.0');
```

**Logic:** Render (and every PaaS) assigns a random port via `PORT`. Hardcoding 3000 is the single most common reason a deployed app returns "service not found". `'0.0.0.0'` makes it listen on all interfaces, not just localhost.

### Step 10.7 — Add a health endpoint

`src/app.controller.ts`:

```typescript
@Public()
@Get("health")
health() {
  return { status: "ok", uptime: process.uptime() };
}
```

**Logic:** the health check lets the platform know the app is alive. It should not require a token, so mark it `@Public()`. Make it public but do not expose version numbers or dependency details.

### Step 10.8 — Verify the deployment

```bash
curl https://your-app.onrender.com/api/health
curl https://your-app.onrender.com/api/docs
curl https://your-app.onrender.com/api/todos
```

Test login against the deployed host:

```bash
curl -X POST https://your-app.onrender.com/api/auth/login -H "Content-Type: application/json" -d '{"email":"a@b.com","password":"password123"}'
```

**Logic:** Render's free tier **sleeps** after inactivity, so the first request after a pause takes ~30s while it wakes up. That is normal on the free plan, not a bug.

### Step 10.9 — Railway (alternative, same idea)

1. Push to GitHub.
2. https://railway.app → **New Project** → **Deploy from GitHub repo**.
3. Railway detects `package.json` and builds it. Set:
   - **Start Command**: `npm run start:prod`
   - **Build Command**: `npm ci && npm run build && npx prisma db migrate`
4. **Variables** tab: add the same env vars.
5. **Settings → Networking → Generate Domain** → you get a public URL.

### Step 10.10 — Deployment checklist

```text
[ ] .env is NOT in git
[ ] JWT_SECRET is new for production (not the dev one)
[ ] DATABASE_URL is the hosted Postgres, not localhost
[ ] app listens on process.env.PORT
[ ] /health exists and is @Public()
[ ] build command runs prisma generate + migrations
[ ] Swagger (/docs) does not leak secrets
[ ] CORS origins are your real front-end, not '*'
[ ] passwords hashed, passwordHash never returned
[ ] guards applied globally (deny by default)
[ ] token expiry is short (15m-1h)
```

### Step 10.11 — What to learn after this

| Topic | Why |
|---|---|
| **Refresh tokens** | access tokens expire; refresh tokens get new ones |
| **Rate limiting** (`@nestjs/throttler`) | stop brute-force on `/auth/login` |
| **Swagger security** | lock down `/docs` in production |
| **Structured logging** (pino) | searchable logs |
| **Docker** | portable builds (you skipped it here on purpose) |
| **CI with `prisma db verify`** | catch drift before deploy |
| **Connection pooling** (PgBouncer) | serverless opens too many connections |
| **Microservices** | split into separate services (Redis transport) |
| **Testing** | unit + e2e (you skipped it here on purpose) |

**Exercise:**
1. Deploy to Render with Neon; hit `/api/health` from another machine.
2. Rotate `JWT_SECRET` in the Render dashboard and confirm existing tokens stop working (401).
3. Add `@nestjs/throttler` and limit `/auth/login` to 5 attempts per minute.

---

## Appendix A: Official Docs& References

| Topic | Link |
|---|---|
| NestJS docs | https://docs.nestjs.com/ |
| NestJS CLI | https://docs.nestjs.com/cli |
| NestJS validation | https://docs.nestjs.com/techniques/validation |
| NestJS guards | https://docs.nestjs.com/security/guards |
| NestJS interceptors | https://docs.nestjs.com/interceptors |
| NestJS exception filters | https://docs.nestjs.com/exception-filters |
| NestJS auth (JWT) | https://docs.nestjs.com/security/authentication |
| NestJS configuration | https://docs.nestjs.com/techniques/configuration |
| NestJS Swagger | https://docs.nestjs.com/openapi/introduction |
| Prisma ORM 8 quickstart | https://www.prisma.io/docs/prisma-orm/quickstart/postgresql |
| Prisma the data contract | https://www.prisma.io/docs/orm/contract-authoring/the-data-contract |
| Prisma contract files | https://www.prisma.io/docs/orm/contract-authoring/the-contract-artifact |
| Prisma reading data | https://www.prisma.io/docs/orm/fundamentals/reading-data |
| Prisma writing data | https://www.prisma.io/docs/orm/fundamentals/writing-data |
| Prisma transactions | https://www.prisma.io/docs/orm/fundamentals/transactions |
| Prisma runtime reference | https://www.prisma.io/docs/orm/reference/transactions-and-runtime |
| Prisma coming from v7 | https://www.prisma.io/docs/orm/coming-from-prisma-orm-7 |
| Prisma 7 to 8 upgrade | https://www.prisma.io/docs/guides/upgrade-prisma-orm/postgresql |
| PostgreSQL docs | https://www.postgresql.org/docs/current/ |
| PostgreSQL download (Windows) | https://www.postgresql.org/download/windows/ |
| class-validator | https://github.com/typestack/class-validator |
| Passport JWT strategy | http://www.passportjs.org/docs/strategies/jwt/ |
| Auth0 validate access tokens | https://auth0.com/docs/secure/tokens/access-tokens/validate-access-tokens |
| Auth0 quickstart | https://auth0.com/docs/quickstart/spa/react |
| Neon docs | https://neon.com/docs |
| Render docs | https://render.com/docs |
| Railway docs | https://docs.railway.com |

---

## Appendix B: Command Reference (Windows CMD & PowerShell)

### B.1 Node / npm

| Purpose | CMD | PowerShell |
|---|---|---|
| node version | `node -v` | `node -v` |
| npm version | `npm -v` | `npm -v` |
| install deps | `npm install` | `npm install` |
| install prod dep | `npm install axios` | `npm install axios` |
| install dev dep | `npm install -D typescript` | `npm install -D typescript` |
| exact install (CI) | `npm ci` | `npm ci` |
| run dev (watch) | `npm run start:dev` | `npm run start:dev` |
| build | `npm run build` | `npm run build` |
| run prod build | `npm run start:prod` | `npm run start:prod` |
| list scripts | `npm run` | `npm run` |
| clear cache | `npm cache clean --force` | `npm cache clean --force` |
| global Nest CLI | `npm install -g @nestjs/cli` | `npm install -g @nestjs/cli` |

### B.2 Nest CLI

| Purpose | Command |
|---|---|
| new project | `nest new simple-api` |
| new project (strict, no git) | `nest new simple-api --skip-git` |
| generate module | `nest g module todos` |
| generate service | `nest g service todos` |
| generate controller | `nest g controller todos` |
| generate guard | `nest g guard jwt-auth` |
| generate interceptor | `nest g interceptor logging` |
| generate filter | `nest g filter all-exceptions` |
| generate pipe | `nest g pipe parse-int` |
| generate everything (CRUD) | `nest g resource todos` |
| spec files | `nest g resource todos --spec false` |

### B.3 Prisma 8

| Purpose | Command |
|---|---|
| init in existing app | `npx prisma orm init --write-env` |
| validate the contract | `npx prisma contract emit` |
| create tables (new DB) | `npx prisma db init` |
| plan a migration | `npx prisma migration plan` |
| apply a migration | `npx prisma db migrate` |
| check DB matches contract | `npx prisma db verify` |
| contract from live DB | `npx prisma contract infer` |
| DB schema as SQL | `npx prisma db schema` |
| apply contract to existing DB | `npx prisma db update` |
| show config | `npx prisma validate` |
| reset dev DB | `npx prisma db reset` |

### B.4 Prisma 7 (fallback)

| Purpose | Command |
|---|---|
| init | `npx prisma init --datasource-provider postgresql` |
| generate client | `npx prisma generate` |
| create + apply migration (dev) | `npx prisma migrate dev --name init` |
| apply migrations (prod) | `npx prisma migrate deploy` |
| browse data | `npx prisma studio` |
| reset dev DB | `npx prisma migrate reset` |

### B.5 PostgreSQL

| Purpose | CMD | PowerShell |
|---|---|---|
| install | `winget install PostgreSQL.PostgreSQL.16` | same |
| version | `psql --version` | `psql --version` |
| is server up? | `pg_isready -h localhost -p 5432` | same |
| connect | `psql -U postgres -h localhost -p 5432` | same |
| start service | `net start postgresql-x64-16` | `Start-Service postgresql-x64-16` |
| stop service | `net stop postgresql-x64-16` | `Stop-Service postgresql-x64-16` |
| list services | `sc query state= all \| findstr postgres` | `Get-Service postgresql*` |
| list DBs | `psql -U postgres -l` | `psql -U postgres -l` |
| run a file | `psql -U postgres -d nestdb -f file.sql` | same |

Inside `psql`:

| Command | Meaning |
|---|---|
| `\l` | list databases |
| `\c nestdb` | connect to a database |
| `\dt` | list tables |
| `\d "Todo"` | describe table |
| `\du` | list roles/users |
| `\dn` | list schemas |
| `\q` | quit |

### B.6 curl / testing

**CMD**

```cmd
curl http://localhost:3000/todos
curl -X POST http://localhost:3000/todos -H "Content-Type: application/json" -d "{\"title\":\"test\"}"
curl -X PATCH http://localhost:3000/todos/ID -H "Content-Type: application/json" -d "{\"completed\":true}"
curl -X DELETE http://localhost:3000/todos/ID
curl http://localhost:3000/todos -H "Authorization: Bearer TOKEN"
```

**PowerShell**

```powershell
Invoke-RestMethod -Uri http://localhost:3000/todos
Invoke-RestMethod -Uri http://localhost:3000/todos -Method POST -ContentType "application/json" -Body '{"title":"test"}'
Invoke-RestMethod -Uri http://localhost:3000/todos/ID -Method PATCH -ContentType "application/json" -Body '{"completed":true}'
Invoke-RestMethod -Uri http://localhost:3000/todos/ID -Method DELETE
Invoke-RestMethod -Uri http://localhost:3000/todos -Headers @{ Authorization = 'Bearer TOKEN' }
```

### B.7 Git

| Purpose | Command |
|---|---|
| init repo | `git init` |
| stage all | `git add .` |
| commit | `git commit -m "message"` |
| branch | `git branch -M main` |
| remote add | `git remote add origin URL` |
| push | `git push -u origin main` |
| see changes | `git status` |
| undo staged | `git restore --staged .` |

---

## Appendix C: Simple Tips & Troubleshooting

### C.1 Ports in this guide

| Service | Port |
|---|---|
| NestJS | 3000 (Render assigns its own in production) |
| PostgreSQL | 5432 |
| Nest with a different port | `npm run start:dev -- --port 4000` |

### C.2 Common errors and fixes

| Error | Cause | Fix |
|---|---|---|
| `EADDRINUSE: address already in use :::3000` | port already taken | `netstat -ano \| findstr :3000` then `taskkill /PID <pid> /F` (PowerShell: `Stop-Process -Id <pid>`), or run on another port |
| `Module './cats.module' not found` | file missing / wrong path | check filename and import path |
| `Cannot resolve dependency` in DI | service not listed in a module's `providers` | add it |
| `Nest can't resolve parameters of JwtStrategy` | type not in `providers` or missing `@Injectable()` | add both |
| 404 on every route | controller not in a module's `controllers`, or module not imported into `AppModule` | check all three levels |
| 400 on every POST | DTO not validated (`ValidationPipe` missing) or DTO not a class | recheck Lesson 6 |
| `PrismaClient is unable to run` / no client generated | `contract emit` not run | `npx prisma contract emit` |
| `P1001 Can't reach database` | Postgres down | `pg_isready -h localhost -p 5432`, start the service |
| `P1000 authentication failed` | bad user/password in `DATABASE_URL` | re-check the string |
| `P2002 unique constraint` | duplicate `@unique` value | catch `sqlState === '23505'` → 409 |
| `relation "Todo_userId_fkey" not found` | migration missing the foreign key | `npx prisma migration plan && npx prisma db migrate` |
| 401 on a valid token | secret mismatch, expired, wrong issuer/audience | compare `JWT_SECRET` in both environments |
| `CONTRACT.MARKER_MISMATCH` warning | DB not migrated to match the contract | `npx prisma db migrate` |
| deployed app returns 502 | listening on 3000 instead of `process.env.PORT` | fix `main.ts` |
| `psql: command not found` | Postgres `bin` not on PATH | add `C:\Program Files\PostgreSQL\16\bin` |
| port 5432 already in use | another Postgres installed | `netstat -ano \| findstr :5432`, or use another port and change `DATABASE_URL` |

### C.3 Where things live

| Thing | Path |
|---|---|
| app bootstrap | `src/main.ts` |
| root module | `src/app.module.ts` |
| Prisma config | `prisma.config.ts` |
| data contract | `src/prisma/contract.prisma` |
| generated types | `src/prisma/contract.json` / `contract.d.ts` |
| DB client | `src/prisma/db.ts` |
| Nest's DB service | `src/prisma/prisma.service.ts` |
| env vars | `.env` (gitignored), `.env.example` (committed) |
| generated Nest files | `src/<feature>/` |
| manual HTTP tests | `requests.http` |

### C.4 Debugging checklist

When something does not work, check in this order:

1. Is the dev server running? (`npm run start:dev` — did it recompile?)
2. Does the URL match the route? (`/api/v1/todos` vs `/todos`)
3. Did you save the file? (watch mode restarts on save)
4. Is the token present and unexpired?
5. Is Postgres up? (`pg_isready`)
6. Is `DATABASE_URL` correct in **this** environment?
7. Did you run `contract emit` + `db migrate` after the model change?
8. Check the server console — the stack trace is usually right there.

### C.5 Good habits from the start

- One feature per module (`todos/`, `auth/`, `users/`).
- Controllers stay thin; services hold logic and database calls.
- Validate every input with a DTO.
- Never return a `passwordHash`, or any internal field, from an endpoint.
- Scope every query by the current user.
- Deny by default (global guard) and opt out with `@Public()`.
- Read config through `ConfigService` and validate at boot.
- `.env` never in git.

### C.6 Concepts you have now covered

```text
NestJS   module, controller, service, provider, DI, decorator, route param,
         body, query, pipe (validation, parse), guard, interceptor, filter,
         exception, decorator factory, custom decorator, metadata/reflector,
         lifecycle hooks, module scope, global module, global guard,
         config module, Swagger, versioning, CORS, global prefix
Prisma8  contract, contract emit, db init, migration plan, db migrate,
         db verify, models, fields, ids, defaults, @unique, @relation,
         include, where, orderBy, limit/offset, select, aggregate,
         create/update/delete + All/Count forms, upsert, transactions,
         connection string, driver URL
Postgres database, schema, table, column, primary key, foreign key,
         unique constraint, NULL, TEXT, SERIAL/identity, ILIKE
Auth     hashing (bcrypt), JWT claims, signing, expiry, strategy, guard,
         RS256, JWKS, issuer, audience, scopes/permissions, RBAC
REST     resources, verbs, status codes, pagination, filtering
Opsenv vars, build/start commands, PORT, health checks, managed DB
```

---