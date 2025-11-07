## Purpose of this repo
Repo to reproduce bug when using EF to store server-side sessions using [Duende BFF framework](https://docs.duendesoftware.com/bff/).

This is a copy of [this official sample](https://github.com/DuendeSoftware/Samples/tree/main/BFF/v3/BlazorWasm) for Blazor WASM.

Basically just changed this line:

```
builder.Services.AddBff()
    .AddServerSideSessions() // Add in-memory implementation of server side sessions
    .AddBlazorServer();
```

To the following:

```
const string connectionString = @"Data Source=(LocalDb)\MSSQLLocalDB;database=BlazorBffApp2;trusted_connection=yes;";
var migrationsAssembly = typeof(Program).GetTypeInfo().Assembly.GetName().Name;

// BFF setup for blazor
builder.Services.AddBff()
    .AddEntityFrameworkServerSideSessions(options => options.UseSqlServer(connectionString, b => b.MigrationsAssembly(migrationsAssembly)))
    .AddBlazorServer();
```

and added ef migrations according to the documentation.

## The issue

Console logs the following error after login or on each full page refresh:

```
fail: Microsoft.AspNetCore.Components.Server.RevalidatingServerAuthenticationStateProvider[0]
      An error occurred while revalidating authentication state
      System.ObjectDisposedException: Cannot access a disposed context instance. A common cause of this error is disposing a context instance that was resolved from dependency injection and then later trying to use the same context instance elsewhere in your application. This may occur if you are calling 'Dispose' on the context instance, or wrapping it in a using statement. If you are using dependency injection, you should let the dependency injection container take care of disposing context instances.
      Object name: 'SessionDbContext'.
         at Microsoft.EntityFrameworkCore.DbContext.CheckDisposed()
         at Microsoft.EntityFrameworkCore.DbContext.get_ContextServices()
         at Microsoft.EntityFrameworkCore.DbContext.get_InternalServiceProvider()
         at Microsoft.EntityFrameworkCore.DbContext.get_ChangeTracker()
         at Microsoft.EntityFrameworkCore.Query.CompiledQueryCacheKeyGenerator.GenerateCacheKeyCore(Expression query, Boolean async)
         at Microsoft.EntityFrameworkCore.Query.RelationalCompiledQueryCacheKeyGenerator.GenerateCacheKeyCore(Expression query, Boolean async)
         at Microsoft.EntityFrameworkCore.SqlServer.Query.Internal.SqlServerCompiledQueryCacheKeyGenerator.GenerateCacheKey(Expression query, Boolean async)
         at Microsoft.EntityFrameworkCore.Query.Internal.QueryCompiler.ExecuteCore[TResult](Expression query, Boolean async, CancellationToken cancellationToken)
         at Microsoft.EntityFrameworkCore.Query.Internal.QueryCompiler.ExecuteAsync[TResult](Expression query, CancellationToken cancellationToken)
         at Microsoft.EntityFrameworkCore.Query.Internal.EntityQueryProvider.ExecuteAsync[TResult](Expression expression, CancellationToken cancellationToken)
         at Microsoft.EntityFrameworkCore.Query.Internal.EntityQueryable`1.GetAsyncEnumerator(CancellationToken cancellationToken)
         at System.Runtime.CompilerServices.ConfiguredCancelableAsyncEnumerable`1.GetAsyncEnumerator()
         at Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.ToListAsync[TSource](IQueryable`1 source, CancellationToken cancellationToken)
         at Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.ToArrayAsync[TSource](IQueryable`1 source, CancellationToken cancellationToken)
         at Duende.Bff.EntityFramework.UserSessionStore.GetUserSessionsAsync(UserSessionsFilter filter, CancellationToken cancellationToken) in /_/bff/src/Bff.EntityFramework/Store/UserSessionStore.cs:line 193
         at Duende.Bff.Blazor.BffServerAuthenticationStateProvider.ValidateAuthenticationStateAsync(AuthenticationState authenticationState, CancellationToken cancellationToken) in /_/bff/src/Bff.Blazor/BffServerAuthenticationStateProvider.cs:line 140
         at Microsoft.AspNetCore.Components.Server.RevalidatingServerAuthenticationStateProvider.RevalidationLoop(Task`1 authenticationStateTask, CancellationToken cancellationToken)
```

## How to run
1. clone
2. build
3. run
