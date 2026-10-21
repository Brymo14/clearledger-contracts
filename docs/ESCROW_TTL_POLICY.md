# Escrow TTL policy

Escrow instance state and persistent records use the shared TTL policy. Every
write to a locked escrow record or its history extends that key when its
remaining lifetime falls below the threshold, up to `TTL_EXTEND_TO`. The
contract instance receives the same extension for writes that change instance
state.

Callers should keep the escrow active while it is needed and should not rely on
an unrelated read to keep arbitrary keys alive. Tests for new write paths must
advance the ledger beyond the threshold and verify that both the locked record
and its history remain readable.
