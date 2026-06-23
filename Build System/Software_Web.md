# ASP.NET Core | Angular | MSSQL | REST APIs | JWT Authentication | Full-Stack Interview Questions
> Targeted at **5+ Years Experienced Engineers** | Includes Follow-up Deep Dive Questions

---

## Table of Contents
- [ASP.NET Core](#aspnet-core)
- [Angular](#angular)
- [MSSQL](#mssql)
- [REST APIs](#rest-apis)
- [JWT Authentication](#jwt-authentication)
- [Full-Stack Scenarios](#full-stack-scenarios)

---

# ASP.NET Core

## 🟢 Basic Level

### Q1. What is ASP.NET Core and how does it differ from ASP.NET Framework?
**Answer Hints:** Cross-platform, lightweight, modular middleware pipeline, built-in DI, no System.Web dependency.

> **Follow-up:** If you had a legacy ASP.NET Framework app, what would be your migration strategy to ASP.NET Core?

---

### Q2. Explain the ASP.NET Core request pipeline. What is middleware?
**Answer Hints:** `Use`, `Run`, `Map`, order matters, short-circuit behavior.

> **Follow-up:** If a middleware throws an unhandled exception, how does it propagate? How would you add global exception handling?

---

### Q3. What is Dependency Injection (DI) in ASP.NET Core? What are the three lifetimes?
**Answer Hints:** `Transient`, `Scoped`, `Singleton` — explain each with real use cases.

> **Follow-up:** What happens if you inject a `Scoped` service into a `Singleton`? Have you faced this issue in production?

---

### Q4. What is `appsettings.json`? How do you manage environment-specific configurations?
**Answer Hints:** `appsettings.Development.json`, `IConfiguration`, environment variables, `ASPNETCORE_ENVIRONMENT`.

> **Follow-up:** How would you manage secrets (API keys, DB passwords) securely without committing to source control?

---

### Q5. What is the difference between `IActionResult` and `ActionResult<T>`?
**Answer Hints:** Type safety, Swagger/OpenAPI support, return types.

> **Follow-up:** When would you use `IActionResult` over `ActionResult<T>` in a large API project?

---

### Q6. What are Tag Helpers in ASP.NET Core MVC?
**Answer Hints:** Razor syntax, HTML-friendly, replaces HTML Helpers.

> **Follow-up:** Have you written a custom Tag Helper? What real-world problem did it solve?

---

### Q7. What is `Program.cs` in ASP.NET Core 6+? How does it differ from earlier versions?
**Answer Hints:** Minimal hosting model, removed `Startup.cs`, top-level statements.

> **Follow-up:** How do you organize a large application that previously used `Startup.cs` after migrating to minimal API style?

---

### Q8. What is Model Binding in ASP.NET Core?
**Answer Hints:** `[FromBody]`, `[FromQuery]`, `[FromRoute]`, `[FromForm]`, `[FromHeader]`.

> **Follow-up:** How does ASP.NET Core resolve binding conflicts when the same parameter name exists in query string and route?

---

### Q9. What are Filters in ASP.NET Core?
**Answer Hints:** Authorization, Resource, Action, Exception, Result filters — pipeline order.

> **Follow-up:** How would you implement a global audit logging filter for all API calls?

---

### Q10. What is `IHostedService` and `BackgroundService`?
**Answer Hints:** Long-running background tasks, `ExecuteAsync`, cancellation tokens.

> **Follow-up:** How would you implement a background job that processes messages from a queue every 30 seconds?

---

## 🟡 Intermediate Level

### Q11. Explain the difference between `AddMvc()`, `AddControllers()`, `AddRazorPages()`, and `AddControllersWithViews()`.
**Answer Hints:** Feature sets, use cases — API-only vs MVC vs Razor Pages.

> **Follow-up:** In a microservices setup, which would you choose and why?

---

### Q12. How does ASP.NET Core handle Model Validation? How do you implement custom validation?
**Answer Hints:** `DataAnnotations`, `IValidatableObject`, `ValidationAttribute`, `FluentValidation` library.

> **Follow-up:** How would you implement cross-field validation — e.g., `EndDate` must be after `StartDate`?

---

### Q13. What is the Minimal API in ASP.NET Core 6+? When would you use it over controllers?
**Answer Hints:** Route handlers, `MapGet`/`MapPost`, less boilerplate, performance.

> **Follow-up:** What are the limitations of Minimal APIs for large enterprise applications?

---

### Q14. How do you implement Caching in ASP.NET Core?
**Answer Hints:** `IMemoryCache`, `IDistributedCache`, `ResponseCaching`, Redis, cache invalidation strategies.

> **Follow-up:** You have a high-traffic endpoint returning the same data for 5 minutes. Walk me through implementing distributed caching with Redis.

---

### Q15. What is Output Caching in ASP.NET Core 7+?
**Answer Hints:** Differs from Response Caching, server-side, `[OutputCache]` attribute.

> **Follow-up:** How would you invalidate output cache when underlying data changes?

---

### Q16. Explain the Repository Pattern and Unit of Work Pattern in ASP.NET Core.
**Answer Hints:** Abstraction over data layer, testability, `DbContext` as Unit of Work.

> **Follow-up:** Is the Repository Pattern an anti-pattern when using EF Core? Justify your answer with real experience.

---

### Q17. How does ASP.NET Core handle CORS? How do you configure it?
**Answer Hints:** `AddCors`, `UseCors`, policy-based, `AllowAnyOrigin` vs specific origins.

> **Follow-up:** You're getting CORS errors in production but not locally. What would be your debugging steps?

---

### Q18. What is Health Check in ASP.NET Core?
**Answer Hints:** `AddHealthChecks`, `/health` endpoint, custom health checks, DB health, third-party libraries.

> **Follow-up:** How would you integrate health checks with Kubernetes liveness and readiness probes?

---

### Q19. What is Rate Limiting in ASP.NET Core 7+?
**Answer Hints:** `AddRateLimiter`, Fixed/Sliding/Token Bucket/Concurrency limiters.

> **Follow-up:** How would you implement per-user rate limiting (100 requests/min per authenticated user)?

---

### Q20. How does ASP.NET Core handle Logging?
**Answer Hints:** `ILogger<T>`, built-in providers, Serilog, structured logging, log levels.

> **Follow-up:** How would you implement structured logging with correlation IDs to trace a single request across microservices?

---

### Q21. What is `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>`?
**Answer Hints:** Singleton vs Scoped vs live reload behavior differences.

> **Follow-up:** When would you use `IOptionsMonitor` over `IOptions`? Give a real-world example.

---

### Q22. How do you implement Global Exception Handling in ASP.NET Core?
**Answer Hints:** `UseExceptionHandler`, `IExceptionHandler` (ASP.NET Core 8), Problem Details, middleware.

> **Follow-up:** How do you return consistent error responses (RFC 7807 Problem Details) across all APIs?

---

### Q23. Explain Routing in ASP.NET Core — Conventional vs Attribute Routing.
**Answer Hints:** `[Route]`, `[HttpGet]`, route constraints, route templates.

> **Follow-up:** How would you version your routes? Compare URL versioning vs header versioning vs query string versioning.

---

### Q24. What is gRPC in ASP.NET Core? When would you use it over REST?
**Answer Hints:** Protocol Buffers, HTTP/2, strongly typed, performance benefits.

> **Follow-up:** In a microservices architecture, how would you decide between gRPC and REST for internal vs external communication?

---

### Q25. How does `HttpClientFactory` work in ASP.NET Core?
**Answer Hints:** Named, Typed, `IHttpClientFactory`, socket exhaustion problem it solves.

> **Follow-up:** What is the `Polly` library and how would you configure retry and circuit breaker policies with `HttpClientFactory`?

---

## 🔴 Advanced Level

### Q26. What is the ASP.NET Core Kestrel server? How do you configure it for production?
**Answer Hints:** Cross-platform, behind reverse proxy (Nginx/IIS), HTTPS configuration, connection limits.

> **Follow-up:** Walk me through deploying ASP.NET Core on Linux with Nginx as a reverse proxy.

---

### Q27. How does ASP.NET Core handle multi-tenancy?
**Answer Hints:** Tenant resolution (subdomain, header, claim), per-tenant DI, per-tenant database connection.

> **Follow-up:** Design a multi-tenant SaaS application where each tenant has an isolated database. How would you manage migrations?

---

### Q28. What are Source Generators in .NET? How can they improve ASP.NET Core performance?
**Answer Hints:** Compile-time code generation, JSON serialization, `System.Text.Json` source gen, no reflection.

> **Follow-up:** How have you used or considered Source Generators to reduce startup time in a large ASP.NET Core app?

---

### Q29. Explain the Mediator Pattern using MediatR in ASP.NET Core.
**Answer Hints:** CQRS, `IRequest`, `IRequestHandler`, `INotification`, pipeline behaviors.

> **Follow-up:** How would you implement cross-cutting concerns like logging, validation, and caching as MediatR pipeline behaviors?

---

### Q30. What is Native AOT in .NET 8? How does it affect ASP.NET Core apps?
**Answer Hints:** Ahead-of-time compilation, no JIT, smaller binaries, cold start improvements, limitations with reflection.

> **Follow-up:** What are the trade-offs of using Native AOT in a complex ASP.NET Core app with heavy use of reflection?

---

### Q31. How do you implement SignalR in ASP.NET Core for real-time communication?
**Answer Hints:** Hubs, connection management, backplane (Redis), groups, scale-out.

> **Follow-up:** How would you scale a SignalR application across multiple servers? What are the trade-offs of using Redis backplane vs Azure SignalR Service?

---

### Q32. What is the difference between `IAsyncEnumerable<T>` and streaming responses in ASP.NET Core?
**Answer Hints:** Server-Sent Events, HTTP streaming, memory efficiency, `yield return`.

> **Follow-up:** How would you stream large data sets from a database to the client without loading everything into memory?

---

### Q33. How do you implement API Versioning in ASP.NET Core?
**Answer Hints:** `Asp.Versioning.Mvc`, URL/Header/QueryString versioning, deprecated versions, sunset headers.

> **Follow-up:** You need to introduce a breaking change in v2 of your API while keeping v1 running. How would you manage shared business logic?

---

### Q34. Explain how ASP.NET Core manages the application lifecycle with `IHostApplicationLifetime`.
**Answer Hints:** `ApplicationStarted`, `ApplicationStopping`, `ApplicationStopped`, graceful shutdown.

> **Follow-up:** How would you ensure in-flight HTTP requests complete before the application shuts down during a Kubernetes rolling update?

---

### Q35. What is Blazor? Compare Blazor Server vs Blazor WebAssembly vs Blazor United.
**Answer Hints:** Rendering modes, latency, SEO, offline support, .NET 8 unified model.

> **Follow-up:** When would you choose Blazor over Angular for a large enterprise application?

---

## 🔵 Expert Level

### Q36. How would you design a high-performance ASP.NET Core API handling 100,000 requests/sec?
**Answer Hints:** Async/await throughout, minimal allocations, `Span<T>`/`Memory<T>`, `ArrayPool`, response compression, HTTP/2.

> **Follow-up:** How do you profile and identify hot paths in an ASP.NET Core application using BenchmarkDotNet and dotnet-trace?

---

### Q37. Explain the internal request processing pipeline at the .NET runtime level in Kestrel.
**Answer Hints:** `PipeReader`/`PipeWriter`, libuv vs managed sockets, connection middleware, `System.IO.Pipelines`.

> **Follow-up:** How does `System.IO.Pipelines` help avoid buffer allocations compared to traditional `Stream`-based I/O?

---

### Q38. How do you implement Domain-Driven Design (DDD) in ASP.NET Core?
**Answer Hints:** Aggregates, Value Objects, Domain Events, bounded contexts, application layer vs domain layer.

> **Follow-up:** How do you publish and handle Domain Events across bounded contexts in a microservices architecture using ASP.NET Core?

---

### Q39. How would you implement Event Sourcing with CQRS in ASP.NET Core?
**Answer Hints:** Event store, projections, snapshots, eventual consistency, replay.

> **Follow-up:** What are the challenges of debugging and querying an event-sourced system in production?

---

### Q40. How does the ASP.NET Core DI container resolve circular dependencies, and how do you avoid them?
**Answer Hints:** Constructor injection cycle detection, design smell, factory pattern, lazy injection.

> **Follow-up:** Describe a real-world scenario where you encountered a circular dependency and how you restructured the code.

---

---

# Angular

## 🟢 Basic Level

### Q1. What is Angular and how does it differ from AngularJS?
> **Follow-up:** Why would you choose Angular over React for a large enterprise application?

### Q2. What is a Component in Angular? Explain the component lifecycle hooks.
**Key Hooks:** `ngOnInit`, `ngOnChanges`, `ngOnDestroy`, `ngAfterViewInit`, `ngAfterContentInit`.
> **Follow-up:** When would `ngOnChanges` fire but `ngOnInit` not? Give a real example.

### Q3. What is the difference between `ngIf` and `[hidden]`?
> **Follow-up:** In a dashboard with 50 dynamic widgets, which approach would you use for better performance?

### Q4. What are Angular Modules (`NgModule`)? What is a Feature Module?
> **Follow-up:** How does lazy loading work with Feature Modules? Walk me through the router configuration.

### Q5. What is Data Binding in Angular? Explain all four types.
**Types:** Interpolation, Property Binding, Event Binding, Two-way Binding.
> **Follow-up:** What are the performance implications of two-way binding with a large form?

### Q6. What is the difference between `*ngFor` `trackBy` with and without it?
> **Follow-up:** You have a list of 10,000 items rendering slowly. How do you optimize it?

### Q7. What are Angular Services? How are they different from Components?
> **Follow-up:** How would you share state between two sibling components using a Service?

### Q8. What is RxJS? What is an Observable?
> **Follow-up:** Explain the difference between `Subject`, `BehaviorSubject`, `ReplaySubject`, and `AsyncSubject`.

### Q9. What is the Angular CLI? Name 5 important commands.
> **Follow-up:** How would you customize the Angular build pipeline using `@angular-builders/custom-webpack`?

### Q10. What is the difference between `ngModel` and Reactive Forms?
> **Follow-up:** For a complex dynamic form with conditional validations, which approach would you choose?

---

## 🟡 Intermediate Level

### Q11. What is Change Detection in Angular? Explain `Default` vs `OnPush` strategies.
> **Follow-up:** How do you manually trigger change detection in `OnPush` components?

### Q12. What are Angular Guards? Explain `CanActivate`, `CanDeactivate`, `CanLoad`, `Resolve`.
> **Follow-up:** Implement a guard that checks if a user has a specific role before accessing a route.

### Q13. What is the `async` pipe? Why is it preferred over manual subscriptions?
> **Follow-up:** What happens if you use the `async` pipe in two places for the same Observable — does it make two HTTP calls?

### Q14. What is `ViewChild` vs `ContentChild` in Angular?
> **Follow-up:** How would you access a child component's method from a parent component?

### Q15. What is Angular Lazy Loading? How does it work with preloading strategies?
> **Follow-up:** Compare `PreloadAllModules` vs custom preloading strategy. When would you use each?

### Q16. What is the `HttpClient` module? How do you handle errors globally?
> **Follow-up:** How would you implement an HTTP interceptor that adds a JWT token and retries on 401 responses?

### Q17. What are Angular Animations?
> **Follow-up:** How do you trigger animations programmatically vs via state changes?

### Q18. What is `Renderer2` and why should you use it instead of direct DOM manipulation?
> **Follow-up:** When is it necessary to use `Renderer2`? What are Server-Side Rendering (SSR) implications?

### Q19. What is Angular Universal (SSR)?
> **Follow-up:** What are the challenges of SSR with third-party libraries that depend on `window` or `document`?

### Q20. How do you implement State Management in Angular?
**Options:** NgRx, Akita, NGXS, simple Service + BehaviorSubject.
> **Follow-up:** When is NgRx overkill? What is the threshold at which you'd introduce it?

### Q21. What is `InjectionToken` in Angular?
> **Follow-up:** How would you use `InjectionToken` to configure a library with environment-specific values?

### Q22. How does Angular handle forms validation — built-in and custom validators?
> **Follow-up:** Implement an async validator that checks if a username is already taken via an API call.

### Q23. What is `ng-content` and Content Projection?
> **Follow-up:** How do you project multiple named slots into a reusable component?

### Q24. What is the difference between `declarations`, `imports`, `exports`, and `providers` in `NgModule`?
> **Follow-up:** What happens if you declare a component in two different modules?

### Q25. How do you optimize Angular bundle size?
**Hints:** Tree shaking, lazy loading, `--prod` build, `source-map-explorer`.
> **Follow-up:** Walk me through your process of reducing an Angular app from 5MB to 1MB bundle size.

---

## 🔴 Advanced Level

### Q26. What are Angular Standalone Components (Angular 14+)?
> **Follow-up:** How do standalone components change the way you structure a large Angular application?

### Q27. What is the new Angular Signals API (Angular 16+)?
> **Follow-up:** How do Signals compare to RxJS Observables? When would you use each?

### Q28. What is `Zone.js`? How does Angular use it for change detection?
> **Follow-up:** How would you run code outside NgZone for performance optimization?

### Q29. How do you implement Micro-Frontend Architecture with Angular?
**Hints:** Module Federation (Webpack 5), `@angular-architects/module-federation`.
> **Follow-up:** How do you handle shared dependencies and version conflicts in a Micro-Frontend setup?

### Q30. How would you implement real-time features (WebSocket/SignalR) in Angular?
> **Follow-up:** How do you handle reconnection logic and missed messages when the WebSocket disconnects?

---

## 🔵 Expert Level

### Q31. How does Angular's compilation work — JIT vs AOT vs Ivy?
> **Follow-up:** What code transformations does the Angular Ivy compiler perform at the component level?

### Q32. How would you design a plugin architecture in Angular where features can be loaded at runtime?
> **Follow-up:** How do you handle routing, shared state, and inter-plugin communication?

### Q33. How do you implement a fully accessible (WCAG 2.1 AA compliant) Angular application?
> **Follow-up:** How do you test accessibility automatically in CI/CD pipeline?

### Q34. How do you profile and debug performance issues in a large Angular application?
**Tools:** Chrome DevTools, Angular DevTools, Profiler.
> **Follow-up:** Describe the most complex performance issue you've resolved in Angular.

### Q35. What is Deferrable Views in Angular 17+?
> **Follow-up:** How do `@defer`, `@loading`, `@error`, and `@placeholder` blocks replace lazy-loaded modules?

---

---

# MSSQL

## 🟢 Basic Level

### Q1. What is the difference between `WHERE` and `HAVING`?
> **Follow-up:** Can you use `HAVING` without `GROUP BY`? What does it do?

### Q2. What are the different types of JOINs in SQL?
> **Follow-up:** When does a `LEFT JOIN` produce the same results as an `INNER JOIN`?

### Q3. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?
> **Follow-up:** How do you recover data after an accidental `TRUNCATE` in production?

### Q4. What are Indexes? What is a Clustered vs Non-Clustered Index?
> **Follow-up:** A table has a clustered index on `Id`. You query frequently on `Email`. What index would you add?

### Q5. What are Primary Key, Foreign Key, Unique, and Check Constraints?
> **Follow-up:** Should Foreign Keys always be enforced at the database level? What are the arguments against?

### Q6. What is a View in SQL Server?
> **Follow-up:** What is an Indexed View (Materialized View)? When would you use it?

### Q7. What is the difference between `CHAR`, `VARCHAR`, and `NVARCHAR`?
> **Follow-up:** What storage and performance implications does choosing `NVARCHAR(MAX)` over `NVARCHAR(200)` have?

### Q8. What are Aggregate Functions in SQL?
> **Follow-up:** How do `NULL` values affect `COUNT(*)` vs `COUNT(column)`?

### Q9. What is `DISTINCT` vs `GROUP BY`?
> **Follow-up:** Can `DISTINCT` cause performance issues? How would you rewrite it?

### Q10. What is a Subquery vs a JOIN? When do you prefer one over the other?
> **Follow-up:** Rewrite a correlated subquery using a JOIN for better performance.

---

## 🟡 Intermediate Level

### Q11. What are CTEs (Common Table Expressions)? How do recursive CTEs work?
> **Follow-up:** Write a recursive CTE to find all employees in a reporting hierarchy under a given manager.

### Q12. What are Window Functions in SQL Server?
**Functions:** `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG`, `LEAD`, `SUM OVER`.
> **Follow-up:** Find the top 3 highest-paid employees per department using window functions.

### Q13. What is a Stored Procedure vs a Function in SQL Server?
> **Follow-up:** Can a function have side effects? What are the restrictions on Table-Valued Functions?

### Q14. What are Triggers in SQL Server? What are their performance implications?
> **Follow-up:** You need to maintain an audit log of all `UPDATE`s to the `Orders` table. Would you use a trigger or an application-level audit? Justify.

### Q15. What is Transaction Management in SQL Server? Explain ACID properties.
> **Follow-up:** What is a deadlock? How do you detect and prevent deadlocks in MSSQL?

### Q16. What is the difference between Optimistic and Pessimistic Concurrency?
> **Follow-up:** How does EF Core implement optimistic concurrency with `rowversion`?

### Q17. What are Execution Plans? How do you read and optimize them?
**Key Concepts:** Table Scan vs Index Seek, Key Lookup, Hash Join vs Nested Loop.
> **Follow-up:** Walk me through a query optimization you did using execution plans.

### Q18. What are SQL Server Isolation Levels?
**Levels:** `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`, `SNAPSHOT`.
> **Follow-up:** What is Read Committed Snapshot Isolation (RCSI)? How does it reduce blocking?

### Q19. What is Query Store in SQL Server?
> **Follow-up:** How do you use Query Store to identify and fix plan regression after a SQL Server upgrade?

### Q20. What is Partitioning in SQL Server?
> **Follow-up:** You have a 2 billion row Orders table. Design a partitioning strategy for it.

### Q21. What is Columnstore Index?
> **Follow-up:** When would you choose a Columnstore Index over a traditional B-Tree index?

### Q22. What is `NOLOCK` hint? Is it safe to use?
> **Follow-up:** What are dirty reads? In which scenarios is `NOLOCK` acceptable?

### Q23. How do you implement Pagination in SQL Server?
**Methods:** `OFFSET-FETCH`, `ROW_NUMBER()`, keyset pagination.
> **Follow-up:** Why is `OFFSET-FETCH` slow for large page numbers? How does keyset pagination solve this?

### Q24. What is Database Normalization? Explain 1NF, 2NF, 3NF, BCNF.
> **Follow-up:** When would you deliberately denormalize a table? Give a real-world example.

### Q25. What is `TempDB` in SQL Server? What workloads stress it?
> **Follow-up:** How do you reduce `TempDB` contention on a busy server?

---

## 🔴 Advanced Level

### Q26. What is Always On Availability Groups in SQL Server?
> **Follow-up:** How do readable secondaries work and what workloads would you offload to them?

### Q27. How does SQL Server In-Memory OLTP work?
> **Follow-up:** What are the limitations of memory-optimized tables? When is it NOT suitable?

### Q28. What is Change Data Capture (CDC) vs Change Tracking in SQL Server?
> **Follow-up:** You need to sync SQL Server changes to Elasticsearch in near-real-time. Which would you use?

### Q29. How do you implement Row-Level Security (RLS) in SQL Server?
> **Follow-up:** Implement RLS for a multi-tenant application where users can only see their own tenant's data.

### Q30. How do you handle large-scale database migrations with zero downtime?
> **Follow-up:** Walk me through an expand-contract pattern for renaming a heavily-used column in production.

---

## 🔵 Expert Level

### Q31. Explain the SQL Server Lock Architecture — Lock types, Lock escalation.
> **Follow-up:** How would you diagnose and fix a lock escalation issue causing blocking on a high-transaction table?

### Q32. How does SQL Server handle statistics and auto-update statistics?
> **Follow-up:** When would stale statistics cause poor query plans? How do you manage statistics on very large tables?

### Q33. Design a database schema for a multi-tenant SaaS application supporting 10,000 tenants.
> **Follow-up:** Compare single database (row-level tenant ID), database-per-tenant, and schema-per-tenant approaches.

### Q34. How does the SQL Server Query Optimizer work?
> **Follow-up:** What are parameter sniffing problems and how do you resolve them in production?

### Q35. How do you implement Temporal Tables in SQL Server?
> **Follow-up:** How do you query historical data from a temporal table? What are the storage trade-offs?

---

---

# REST APIs

## 🟢 Basic Level

### Q1. What is REST? What are the 6 constraints of REST?
> **Follow-up:** Is every HTTP API a RESTful API? What makes an API truly RESTful?

### Q2. What are HTTP Methods? What is the difference between `PUT` and `PATCH`?
> **Follow-up:** Is `GET` always idempotent? Can a `GET` have a body?

### Q3. What are HTTP Status Codes? Give examples for 2xx, 3xx, 4xx, 5xx.
> **Follow-up:** When should you return `422 Unprocessable Entity` vs `400 Bad Request`?

### Q4. What is the difference between stateless and stateful APIs?
> **Follow-up:** How do you maintain user session in a stateless REST API?

### Q5. What is Content Negotiation in REST?
> **Follow-up:** How do you implement content negotiation in ASP.NET Core (`Accept` header, `produces`)?

### Q6. What is HATEOAS in REST?
> **Follow-up:** Should every REST API implement HATEOAS? What are the practical trade-offs?

### Q7. What is an API endpoint? How do you design RESTful resource URLs?
> **Follow-up:** Is `/getUsers` RESTful? How would you redesign it?

### Q8. What is the difference between REST and SOAP?
> **Follow-up:** In which scenarios would you still choose SOAP over REST in 2024?

### Q9. What is a Query String vs a Path Parameter? When do you use each?
> **Follow-up:** Design the URL structure for filtering, sorting, and paginating a list of products.

### Q10. What is an OpenAPI/Swagger specification?
> **Follow-up:** How do you generate a Swagger spec from ASP.NET Core and publish it?

---

## 🟡 Intermediate Level

### Q11. What is API Versioning? Compare different strategies.
> **Follow-up:** How do you communicate API deprecation to clients?

### Q12. How do you implement Pagination, Filtering, and Sorting in a REST API?
> **Follow-up:** Design the response envelope for a paginated API including metadata.

### Q13. What is CORS? How do you configure it for a REST API?
> **Follow-up:** You allow `*` in CORS for development. Why is this dangerous in production?

### Q14. What is Idempotency? Which HTTP methods are idempotent?
> **Follow-up:** How do you implement idempotency keys for payment APIs?

### Q15. What is ETag and conditional requests in REST?
> **Follow-up:** How do you implement optimistic concurrency control in a REST API using ETags?

### Q16. How do you design a REST API for bulk operations?
> **Follow-up:** Design a bulk create endpoint for 10,000 records. How do you handle partial failures?

### Q17. What is the Richardson Maturity Model?
> **Follow-up:** Most real-world APIs are at Level 2. What prevents teams from reaching Level 3 (HATEOAS)?

### Q18. How do you handle long-running operations in REST APIs?
> **Follow-up:** Implement an async pattern where a client polls for job status after starting a long operation.

### Q19. How do you document a REST API effectively?
> **Follow-up:** What information should be included beyond just the endpoint signature?

### Q20. What is GraphQL? How does it differ from REST?
> **Follow-up:** When would you introduce GraphQL alongside an existing REST API? What are the migration challenges?

---

## 🔴 Advanced Level

### Q21. How do you design a REST API for eventual consistency in a distributed system?
> **Follow-up:** How do you handle a scenario where an order is created but inventory deduction fails in a separate service?

### Q22. What is the Saga Pattern? How does it apply to REST APIs?
> **Follow-up:** Compare Choreography-based vs Orchestration-based Sagas.

### Q23. How do you implement circuit breaker pattern in REST API calls?
> **Follow-up:** Describe a production incident where a circuit breaker saved your system from cascading failures.

### Q24. How do you secure a public REST API against abuse?
> **Follow-up:** Design a rate limiting strategy for a free vs paid tier API.

### Q25. How do you implement backward-compatible API changes?
> **Follow-up:** A new required field is added to a request body. How do you make it backward compatible?

---

## 🔵 Expert Level

### Q26. Design a REST API gateway for a microservices architecture.
> **Follow-up:** How do you handle authentication, rate limiting, request routing, and response aggregation at the gateway?

### Q27. How do you implement API observability (metrics, tracing, logging)?
> **Follow-up:** How do you correlate a slow API response to a specific database query in distributed tracing?

### Q28. Design a REST API that handles 1 million requests per day with SLA of 99.99% uptime.
> **Follow-up:** How does your design change if it needs to handle 1 billion requests per day?

---

---

# JWT Authentication

## 🟢 Basic Level

### Q1. What is JWT? What are its three parts?
**Answer:** Header, Payload, Signature.
> **Follow-up:** Can the payload of a JWT be decoded without the secret? What are the security implications?

### Q2. What is the difference between Authentication and Authorization?
> **Follow-up:** How does JWT handle both authentication and authorization?

### Q3. What claims are typically in a JWT payload?
**Standard:** `sub`, `iss`, `exp`, `iat`, `aud`, `jti`.
> **Follow-up:** What is the `jti` (JWT ID) claim used for?

### Q4. How do you validate a JWT in ASP.NET Core?
> **Follow-up:** What happens if the JWT secret key is rotated? How do you handle token validation during key rotation?

### Q5. What is the difference between `HS256` and `RS256` signing algorithms?
> **Follow-up:** In a microservices architecture, why is `RS256` preferred over `HS256`?

### Q6. Where should you store a JWT on the client side?
**Options:** `localStorage`, `sessionStorage`, `HttpOnly Cookie`.
> **Follow-up:** What are the XSS and CSRF trade-offs for each storage option?

### Q7. What is a Refresh Token? How does it differ from an Access Token?
> **Follow-up:** What is Refresh Token Rotation and why is it important?

### Q8. How do you implement JWT Authentication in ASP.NET Core?
> **Follow-up:** Walk me through the complete flow from login to accessing a protected endpoint.

### Q9. What is the `[Authorize]` attribute in ASP.NET Core?
> **Follow-up:** How do you apply policy-based authorization using JWT claims?

### Q10. What is token expiry (`exp`)? What is a reasonable expiry time for access tokens?
> **Follow-up:** A user's access token expires every 15 minutes. How do you silently refresh it in an Angular app?

---

## 🟡 Intermediate Level

### Q11. How do you implement Role-Based Access Control (RBAC) with JWT?
> **Follow-up:** What are the drawbacks of storing roles in JWT when roles change frequently?

### Q12. What is Claims-Based Identity in ASP.NET Core?
> **Follow-up:** How do you transform external claims (from a third-party IdP) into your application's internal claims?

### Q13. How do you revoke a JWT before it expires?
> **Follow-up:** Design a token revocation system that scales to millions of users.

### Q14. What is OAuth 2.0? How does JWT relate to it?
**Flows:** Authorization Code, Client Credentials, PKCE.
> **Follow-up:** Which OAuth 2.0 flow is appropriate for an Angular SPA + ASP.NET Core API?

### Q15. What is OpenID Connect (OIDC)?
> **Follow-up:** What is the difference between the `access_token` and `id_token` in OIDC?

### Q16. How do you implement Multi-Factor Authentication (MFA) with JWT-based APIs?
> **Follow-up:** How do you handle the intermediate state between first factor success and second factor verification?

### Q17. What is a JWT audience (`aud`) claim? Why is it important?
> **Follow-up:** You have multiple microservices. How do you use the `aud` claim to prevent token misuse?

### Q18. How do you handle JWT authentication in a microservices architecture?
> **Follow-up:** Should each microservice validate the JWT independently, or should only the API Gateway do it?

### Q19. What is `ASP.NET Core Identity`? How does it relate to JWT?
> **Follow-up:** Can you use JWT without ASP.NET Core Identity? What do you lose?

### Q20. What are the security risks of using JWT?
> **Follow-up:** The `alg:none` vulnerability in JWT — explain it and how ASP.NET Core prevents it.

---

## 🔴 Advanced Level

### Q21. How do you implement a secure token refresh mechanism in Angular + ASP.NET Core?
> **Follow-up:** How do you handle multiple parallel requests that all get a 401 and need to refresh — avoiding multiple refresh calls?

### Q22. How do you implement Proof of Possession (PoP) tokens?
> **Follow-up:** How does DPoP (Demonstrating Proof of Possession) prevent token theft?

### Q23. How do you integrate with an external Identity Provider (Azure AD, Okta, Auth0)?
> **Follow-up:** How do you map external IdP groups/roles to your application's authorization policies?

### Q24. What is Token Binding? Why was it abandoned?
> **Follow-up:** What alternatives exist to prevent token theft in transit today?

### Q25. How do you implement fine-grained authorization (Attribute-Based Access Control)?
> **Follow-up:** Design an authorization model where users can access only resources they own or have been explicitly granted access to.

---

## 🔵 Expert Level

### Q26. Design a complete authentication system for a large-scale application (10M+ users).
**Consider:** Token storage, revocation, refresh rotation, MFA, social login, brute force protection.
> **Follow-up:** How does your design change for a zero-trust security model?

### Q27. How do you implement JWT in a federated identity scenario with multiple organizations?
> **Follow-up:** How do you handle claim conflicts when users have identities in multiple organizations?

### Q28. How would you audit and monitor JWT token usage for security anomalies?
> **Follow-up:** Describe how you'd detect and respond to a stolen refresh token being used from a different IP/device.

---

---

# Full-Stack Scenarios

## 🟡 Intermediate Scenarios

### S1. Build a User Registration & Login System
**Stack:** Angular + ASP.NET Core + MSSQL + JWT
> **Follow-up:** How do you handle email verification, password reset, and account lockout?

### S2. Design a Real-Time Dashboard
**Stack:** Angular + SignalR + ASP.NET Core + MSSQL
> **Follow-up:** How do you handle 10,000 concurrent users viewing the same dashboard?

### S3. Implement a File Upload System
> **Follow-up:** How do you handle large file uploads (>1GB) with progress tracking and resume support?

### S4. Build a Role-Based Admin Panel
> **Follow-up:** How do you prevent privilege escalation attacks in both frontend and backend?

### S5. Implement Audit Logging Across the Full Stack
> **Follow-up:** How do you ensure audit logs are tamper-proof?

---

## 🔴 Advanced Scenarios

### S6. Design a Multi-Tenant SaaS Application
> **Follow-up:** How do you handle tenant-specific customizations without code duplication?

### S7. Build a Shopping Cart with Eventual Consistency
> **Follow-up:** How do you handle inventory reservation and order confirmation race conditions?

### S8. Implement a Notification System (Email, SMS, Push)
> **Follow-up:** How do you ensure at-least-once delivery and handle duplicate notifications?

### S9. Design a Reporting Module with Large Dataset Export
> **Follow-up:** How do you generate a 1 million row Excel export without timing out or running out of memory?

### S10. Build a Full-Text Search Feature
> **Follow-up:** How do you implement typeahead search that scales to millions of records?

---

## 🔵 Expert Scenarios

### S11. Migrate a Monolith to Microservices — Full Stack
> **Follow-up:** How do you ensure data consistency when splitting a shared SQL Server database?

### S12. Design a Zero-Downtime Deployment Pipeline
> **Follow-up:** How do you handle database migrations during a rolling deployment?

### S13. Build a CQRS + Event Sourcing System
> **Follow-up:** How do you rebuild a projection that got corrupted in production?

### S14. Design for 99.99% Uptime — Full Stack Architecture
> **Follow-up:** What is your RTO and RPO? How do they influence your architecture decisions?

### S15. Implement Cross-Origin, Multi-Device Authentication
> **Follow-up:** How do you handle session invalidation when a user logs out from one device?

---

---

## 📌 Quick Reference: Stake for Each Level

| Level | Stakes | Focus |
|-------|--------|-------|
| 🟢 Basic | Junior to Mid screening | Core concepts, syntax, fundamentals |
| 🟡 Intermediate | Mid-level validation | Real-world application, trade-offs |
| 🔴 Advanced | Senior validation | Design decisions, performance, scalability |
| 🔵 Expert | Tech Lead / Architect | System design, distributed systems, deep internals |

---

## 📚 Recommended Preparation Resources

- **ASP.NET Core:** Microsoft Docs, Andrew Lock's blog, Steve Smith (Ardalis)
- **Angular:** Angular.io, Decoded Frontend, Angular In Depth
- **MSSQL:** Brent Ozar's blog, SQL Server Central, Itzik Ben-Gan's T-SQL books
- **REST APIs:** Roy Fielding's dissertation, "REST API Design Rulebook"
- **JWT/Security:** OWASP, RFC 7519, Auth0 blog, Philippe De Ryck
- **System Design:** "Designing Data-Intensive Applications" — Martin Kleppmann

---

*Good luck with your interview preparation! Focus on the follow-up questions — they reveal real-world experience.*