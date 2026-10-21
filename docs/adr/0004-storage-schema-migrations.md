# ADR 0004: Storage schema migration markers

## Status

Accepted

## Context

Soroban instance storage is persistent across contract upgrades. Changing the
shape of a stored value without recording the schema that produced it makes an
upgrade ambiguous and can cause a new implementation to decode legacy bytes as a
different type.

## Decision

Each contract that changes a persistent key or value must reserve a `SchemaVersion`
instance key and expose a read-only `storage_version()` method. The value starts at
`1` for existing deployments. Upgrade code must read the current version, migrate
one version at a time, and write the new version only after every conversion has
completed. Unknown future versions must fail closed rather than being interpreted
as the newest schema.

Migration tests must cover a freshly initialized contract, each supported legacy
version, and an unknown version. Tests should assert both the converted value and
the version marker, including that a failed migration leaves the old marker intact.

This marker is independent of the event catalog version in `docs/EVENTS.md`; event
compatibility and storage compatibility can therefore be reviewed separately.

## Consequences

Future storage changes carry an explicit migration boundary and can be rolled out
without guessing which layout a deployed contract contains. Existing contracts
remain on version `1` until a concrete schema migration is introduced.
