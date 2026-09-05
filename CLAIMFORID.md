# `ClaimForId` Amount Validation

`Bolt11ClaimForIdRequest.claimable_amount_msat` is optional. When omitted, ldk-server passes `u64::MAX` to LDK Node. LDK Node only rejects when the caller-supplied amount is lower than its stored expectation; it does not verify that the supplied amount equals the amount actually held. Consequently, an omitted or excessively large value passes validation, while `claim_funds` settles only the real HTLC amount.

The parameter originated in LDK Node as a defensive check: callers were expected to copy the actual `claimable_amount_msat` from `PaymentClaimable`, compare it with their business expectation, and claim or fail accordingly. ldk-server later made the field optional as an unchecked convenience.

However, ldk-server currently discards LDK Node's authoritative `PaymentClaimable.claimable_amount_msat`. The nested `payment.amount_msat` may match it for current non-JIT payments, but can represent a gross, expected, or legacy amount in other cases. Clients therefore cannot reliably perform the intended validation.

Recommended fix: expose and retain the authoritative claimable amount by payment ID. Either remove the amount from `ClaimForId` and use the retained value internally, or interpret the request value as a minimum acceptable amount and compare it against the retained actual amount. Remove the `u64::MAX` fallback.
