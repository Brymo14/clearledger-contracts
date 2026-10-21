# Pool factory resource budgets

`register_asset` deploys and initializes a pool, so its resource use grows with
the number of already registered assets only through the bounded registry and
mapping lookups. Resource measurements should record CPU instructions and
memory bytes separately for realistic asset counts (1, 10, 25, and 50).

The 50 asset case is the supported operational bound for a single invocation.
Deployments should measure the complete `register_asset` call on the target
Soroban protocol version and keep a margin below the transaction limits. A
failed measurement must not be used to raise the bound; reduce the batch size
instead.
