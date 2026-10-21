# Pool funding asset naming

Pool instances are asset neutral. `get_funding_asset` is the canonical getter
for the asset held by a pool and should be used by new integrations, regardless
of whether the asset is USDC or another Stellar asset.

`get_usdc_asset` remains available as a compatibility alias for deployed
clients. It returns the same configured address and does not imply that a pool
only accepts USDC. New documentation and client code must use
`get_funding_asset`.
