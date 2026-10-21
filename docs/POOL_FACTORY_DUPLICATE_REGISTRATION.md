# Duplicate asset registration

`register_asset` is deliberately non-idempotent: registering an asset that
already has a pool returns the typed `AssetAlreadyRegistered` error. The
existing pool mapping is never replaced and no second pool is deployed.

Clients that may retry a request should first call `get_pool_for_asset` and
treat an existing mapping as success. They should surface
`AssetAlreadyRegistered` when the mapping is unexpectedly absent from their
local view so an operator can reconcile state rather than silently selecting a
different pool.
