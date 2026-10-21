# Registry Verification Lifecycle and Revocation

This page describes how participant profiles move through the registry in
`contracts/registry/src/lib.rs`: registration, admin verification, revocation,
and reinstatement. It also states what revocation does to invoices that were
already funded. The behavior below is the behavior of this checkout on Stellar
Testnet. Nothing here implies Mainnet readiness or guaranteed financing or
returns.

## Lifecycle

1. **Registration.** `register_issuer` and `register_buyer` (and the batch
   variants `batch_register_issuers` / `batch_register_buyers`) create a
   `Profile` keyed by the participant's address. Registration requires the
   participant's own authorization and starts the profile **unverified**.
   Registration appends the address to the per-role enumeration index and
   increments `ProfileCount(role)`.
2. **Admin verification.** `verify_profile(address, true)` sets the profile's
   verified flag, clears its revoked flag, and emits `profile_verified`. Only
   the stored contract admin (`DataKey::Admin`) may call it.
3. **Revocation.** `revoke(address)` sets `verified = false` and
   `revoked = true` and emits `address_revoked`. `verify_profile(address,
   false)` achieves the same verification outcome but emits
   `profile_verified` instead. Revocation requires the stored admin's
   authorization.
4. **Reinstatement.** `reinstate(address)` sets `verified = true` and
   `revoked = false` and emits `address_reinstated`. It is the admin-only
   inverse of `revoke`; the dedicated event lets indexers distinguish a
   reinstatement from an ordinary verification.

`get_verification_status(address)` reports `Unregistered`, `Pending`,
`Verified`, or `Revoked`. `is_verified(address)` is the boolean used by other
contracts and returns `false` for both missing profiles and revoked profiles.

### Revocation is not deletion

Revocation only flips flags on the existing `Profile` entry. It does **not**
delete the profile, remove its `ProfileIndex(role, n)` slot, decrement
`ProfileCount(role)`, or erase metadata. Revoked profiles therefore remain
enumerated by `get_profile_count` and `list_profiles`, and `get_profile` still
returns them. See [STORAGE.md](STORAGE.md) for the profile layout and index
keys.

### Edge cases in the current code

- `revoke` is idempotent for an already-revoked profile: it returns `true`
  without rewriting storage or re-emitting the event.
- `revoke` on a registered but never-verified profile returns `true` as a
  no-op and leaves the profile in `Pending` state, because the early return
  keys off the verified flag. Use `verify_profile(address, false)` when an
  unverified profile must explicitly become `Revoked`.
- Revocation and reinstatement are state-changing calls, so they revert while
  the registry is paused. Read-only views such as `is_verified` and
  `get_verification_status` remain callable. See
  [EVENTS.md](EVENTS.md) for the emitted events and
  [LIMITATIONS.md](LIMITATIONS.md) for known constraints.

### Admin authorization and transfer

The admin address is stored under `DataKey::Admin` at `initialize(admin)`.
`verify_profile`, `revoke`, `reinstate`, `pause`, and `transfer_admin` all
require that stored admin's authorization. `transfer_admin(new_admin)` moves
the role with only the current admin's signature, whereas
`transfer_ownership(new_admin)` requires both the current and incoming admin.
After either transfer, only the new admin can verify, revoke, or reinstate
profiles.

## Effect on invoices

Registry verification is checked at these points:

- `contracts/invoice/src/lib.rs`: `create` requires both issuer and buyer to be
  verified; `list_for_financing` (and each `batch_list_for_financing` entry)
  re-checks both before moving an invoice to `Listed`.
- `contracts/pool/src/lib.rs`: `fund_invoice` (and `batch_fund_invoice`)
  re-checks both through `is_verified` immediately before pool capital is
  committed.

Consequences for revocation:

- **Before funding.** A revocation between `create` and listing blocks
  `list_for_financing`. A revocation between listing and funding blocks
  `fund_invoice` with `IssuerNotVerified` or `BuyerNotVerified`; the invoice is
  not automatically delisted by the registry, but funding will fail.
- **After funding.** Once funding has succeeded, verification is deliberately
  not re-checked at any later lifecycle step (`mark_shipped`,
  `confirm_delivery`, `repay`, `repay_early`, or `trigger_default`). A later
  revocation does **not** unwind, refund, default, or otherwise change an
  invoice that was already funded; pool liquidity is already committed and the
  repayment terms are fixed. The invoice continues through its normal
  repayment path, and revocation grants no right to recover funds.

Funding is permissionless and revocation is admin-only; neither path promises
financing or returns. These contracts are Testnet-only and have not been
formally audited.
