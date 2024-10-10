# cloudflare-workers-prisma-connection

If you are using a DB where external connections are maintained, such as PostgreSQL, using Prisma instances around will cause errors.

https://cloudflare-workers-prisma-connection.mofon001.workers.dev/

```ts
export * from '@prisma/adapter-pg';
import { Pool } from 'pg';
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient } from '@prisma/client';

export interface Env {
	DATABASE_URL: string;
	prisma: Fetcher;
}

// Errors occur when Prisma instances are used around.
let prisma: PrismaClient;

export default {
	async fetch(request: Request, env: Env, _ctx: ExecutionContext): Promise<Response> {
		const url = new URL(request.url);
		if (url.pathname !== '/') return new Response('Not Found', { status: 404 });
		if (!prisma) {
			const databaseUrl = new URL(env.DATABASE_URL);
			const schema = databaseUrl.searchParams.get('schema') ?? undefined;
			const pool = new Pool({
				connectionString: env.DATABASE_URL,
			});
			const adapter = new PrismaPg(pool, { schema });
			prisma = new PrismaClient({ adapter });
		}
		await prisma.test.create({ data: {} });
		const result = await prisma.test.findMany().then((r) => r.map(({ id }) => id));
		return new Response(JSON.stringify(result, undefined, '  '), {
			headers: { 'content-type': 'application/json' },
		});
	},
};
```
