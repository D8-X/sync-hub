# Go Project Guidelines

## Scaffolding Convention
- By default, we use the server project layout: https://go.dev/doc/modules/layout#server-project
- Each package should normally have a `<package>.go` file containing its primary public API or main type. Split implementation details into files named after their responsibility.
- Environment variables are defined in a package called `env`, with a type `type EnvKey string`, and defined as constants such as `const RedisPW EnvKey = "REDIS_PW"`.
- Read the environment once during startup, validate it, and then make the rest of the application unaware that the values stem from environment variables. We typically perform this housekeeping in `svc.go`; sometimes we also load AWS secrets there.
- Create slim files for `cmd/<service>/main.go` that call their run function in package `svc`. Package `svc` loads and validates configuration, constructs the required components, and runs the service.
- On missing required configuration, error out. Don't assume default values or start a partially configured service.
- Dockerfiles sit in the cmd directory. Docker-compose in the project root.

## Practical Tips
- Build the smallest complete solution first: the minimum solution that solves the current problem. Prioritize the engine and wheels before the paint and windshield wipers. Avoid placeholder code: solve it properly or leave it out.
- Error flow: handle errors early and keep the happy path minimally indented.
- Errors: don't discard errors. Don't log an error and continue unless recovery is intentional. At lower levels, handle or return them; log errors at service boundaries to avoid logging the same error multiple times. Error strings start lowercase and have no trailing punctuation.
- Context: pass `context.Context` as the first parameter; don't store it in structs.
- Concurrency: every goroutine should have an obvious lifetime and exit condition. Prefer synchronous APIs where possible. Use channels for communication, coordination, and ownership transfer between goroutines; use mutexes where they are the simpler way to protect shared state. Unbuffered channels can be used for synchronization.
- Package naming: package names supply context, so it is best not to repeat them in exported identifiers: `redis.Client`, not `redis.RedisClient`; `order.Request`, not `order.OrderRequest`.
- In the spirit of Go, choose function and variable names that are short, clear, contextual, and non-redundant. Accordingly, don't use multiple names for the same thing.
- Long functions are often a sign that the function has more than one responsibility. Split functions along logical boundaries, not merely to reduce line count.
- Prefer concrete types. Introduce an interface when a consumer actually needs one; do not define interfaces speculatively or merely for mocking.
- When typing values, the compiler can help identify issues. For example, use a type for `"BUY"` and `"SELL"` rather than plain strings. Another example is the `env` pattern described above.
- No packages named `utils` or `helpers`; name the package after what it does.
- Formatting: all Go code must be formatted with `gofmt` or `goimports`. There is no fixed line-length limit. If a line becomes uncomfortable to read, prefer simplifying the code over wrapping solely to satisfy a column limit.

Many of these points are either directly from or in the spirit of https://go.dev/wiki/CodeReviewComments.

## Redis
- We use Ruedis.
- Implement the code compatible with valkey (open source version) and serverless deployments. A major difference is the presence of shards. 
- Centralize Redis operations and key definitions in package `redis`; don't scatter Redis key strings or direct Redis operations throughout the application. 

## APIs
- Check inputs before resource-intensive calls; error out early.
- Avoid DB actions and on-chain queries in the hot path as much as possible.
- Cluster all routes into a file called `routes.go`, so a new developer can easily identify all routes.

## Database
- Put migrations in `db/migrations` as SQL files.
- Before a migration has been deployed anywhere shared, it may be edited freely, that is do not create migration files during development stage so the db schema can easily be ready without going through migrations. Once deployed, create a new migration for each change.
- Keep database access behind clearly identifiable packages. Prefer queries close to the domain or component that owns them; avoid SQL scattered throughout service logic.
