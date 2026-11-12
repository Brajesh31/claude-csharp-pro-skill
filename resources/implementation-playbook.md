# C# Implementation Playbook

## Project conventions

Before changing code, identify the target framework, nullable-reference-type setting, language version, dependency-injection pattern, logging system, and test framework. Follow the repository's existing conventions unless they are demonstrably unsafe.

## Implementation checklist

1. Model domain concepts with focused types and clear invariants.
2. Prefer immutable records for data-transfer values where appropriate.
3. Use nullable annotations to express absence, not as decoration.
4. Use `async` and `await` for I/O-bound work.
5. Pass `CancellationToken` through asynchronous layers.
6. Dispose `IDisposable` and `IAsyncDisposable` resources deterministically.
7. Validate external input at system boundaries.
8. Use structured logging and avoid logging secrets or sensitive personal data.
9. Add unit and integration tests for success, failure, and cancellation.

## Async example

```csharp
public sealed class CustomerService(HttpClient httpClient)
{
    public async Task<Customer?> GetAsync(
        string customerId,
        CancellationToken cancellationToken)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(customerId);

        using var response = await httpClient.GetAsync(
            $"customers/{Uri.EscapeDataString(customerId)}",
            cancellationToken);

        if (response.StatusCode == HttpStatusCode.NotFound)
            return null;

        response.EnsureSuccessStatusCode();

        return await response.Content.ReadFromJsonAsync<Customer>(
            cancellationToken: cancellationToken);
    }
}
```

Register `HttpClient` through `IHttpClientFactory` in applications that need managed connection lifetimes and configured policies. Add retries only for transient failures and only when the operation is safe to retry.

## Common failure patterns

- Blocking asynchronous work with `.Result` or `.Wait()`.
- Using `async void` outside event handlers.
- Creating a new `HttpClient` for every request.
- Catching exceptions without adding context or handling them.
- Using `Task.Run` to disguise synchronous I/O.
- Forgetting cancellation or disposal.
- Returning mutable internal collections directly.

## Review gates

- Build with nullable warnings enabled.
- Run analyzers and format checks used by the project.
- Test cancellation and error paths.
- Verify authorization at service boundaries.
- Confirm that logs do not expose tokens, credentials, or personal data.
- Check performance with realistic input sizes before adding caching or concurrency.
