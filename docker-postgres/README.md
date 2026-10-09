# Running PostgreSQL on Docker + Prisma ORM

This trick or method can save a lot of time when building something for local use or fun projects, especially when you're not focusing on scalability.

## Steps

### 1. Set up the Turborepo

Start a Bun Turborepo:

```bash
bunx create-turbo@latest your-app-name
```

### 2. Set up the DB package

* Inside `packages`, create a `db` folder.
* Run `bun init` inside that folder.
* Follow the **Prisma v7 documentation**.
* Install the required dependencies inside the `db` folder.
* Initialize Prisma using:

```bash
bunx prisma init
```

**Note:** Don't run `bunx create-db`, as it will create a Prisma database URL that we don't want.

Also, don't worry about the database URL mentioned in the Docker section yet.

Create a rough schema for anything in your Prisma schema file.

## The Docker Part

Make sure the PostgreSQL Docker image is pulled on your machine.

Initialize a PostgreSQL container:

```bash
docker run --name pm-postgres -p 5433:5432 -e POSTGRES_PASSWORD=postgres -d postgres
```

You can research how these Docker configurations work from scratch, including why we use `5433:5432` instead of `5432:5432`.

If the container already exists, simply start it using:

```bash
docker start pm-postgres
```

After this, add the local PostgreSQL URL to your `.env` file. Make sure you've installed the required environment variable dependencies if needed.

```env
DATABASE_URL="postgres://postgres:postgres@127.0.0.1:5433/your_db_name"
```

Replace `your_db_name` with the database name you want to use.

## Final Run (Testing)

Create a simple database schema in your Prisma schema file.

Then run the following commands to see it in action:

```bash
bunx prisma migrate dev
bunx prisma generate
bunx prisma studio
```

And there you go! You now have PostgreSQL running locally through Docker, connected to Prisma ORM.

## Upcoming

In the upcoming commit, we'll see how to connect the generated schema files to the backend by exporting them.

## Integrating into the Backend

Simply copy the `db.ts` file from the Prisma + Bun v7 documentation guide and paste it into the `index.ts` file inside the `db` folder.

And there you go! Our generated Prisma client is ready to be exported.

Just a few more steps.

### 1. Export the DB Package

Add the following to the `package.json` file inside the `db` folder:

```json
"exports": {
  "./": "./index.ts"
}
```

**Important:** Use the `exports` keyword, not `export`. Believe me, it took me almost 20 minutes to figure out what was wrong, so consider this an early warning!

### 2. Initialize the Backend

Before importing the DB package, make sure your backend is initialized.

Inside the backend folder, run:

```bash
bun init .
bun add express
bun add -d @types/express
```

### 3. Add the DB Dependency

In the backend's `package.json`, add the DB package to `peerDependencies`:

```json
"peerDependencies": {
  "db": "*"
}
```

Now, run `bun install` from the root of your Turborepo to install the dependencies.

### 4. Import Prisma into the Backend

You can now use the Prisma client in your backend like this:

```typescript
import { prisma } from "db";
```

That's it! Your backend can now access the Prisma client exported from your DB package.