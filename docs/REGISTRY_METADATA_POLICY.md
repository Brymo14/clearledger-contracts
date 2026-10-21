# Registry profile metadata policy

Profile metadata is bounded before every registration, batch registration, and
update. A map may contain at most 20 entries; keys may be at most 64 bytes and
values at most 512 bytes. Empty keys and values are rejected with
`RegistryError::InvalidMetadata`.

The same validation is applied to every write path, including issuer and buyer
batch operations, so a caller cannot bypass the storage budget by choosing a
different entry point. Clients should validate these limits before submitting a
transaction and treat the typed error as a request correction rather than a
retryable failure.
