---
name: Double Ratchet initReceiver initial DHs
description: Why the responder must pass their signed prekey pair to initReceiver(), and what happens if they don't.
---

## Rule
`DoubleRatchet.initReceiver(sharedSecret, initialDHsKP)` **must** receive the responder's initial ratchet key pair as the second argument. For the Signal Protocol responder (Bob), this is his signed prekey pair (`bobSPK` from `generateSignedPreKey`).

**Why:** The Signal Protocol spec defines Bob's initial state as `DHs = bob_signed_prekey_pair`. The first message Alice sends contains her ratchet public key, which causes `_dhRatchet()` to be called immediately. `_dhRatchet` computes `DH(this.DHs.priv, this.DHr)` — if `DHs` is null this crashes with `Cannot read properties of null (reading 'priv')`.

**How to apply:** When constructing a session for the receiver role, always pass the initial DHs:
```js
const dr = new DoubleRatchet();
dr.initReceiver(sharedBob, bobSPK);  // bobSPK = { pub, priv, keyId, signature }
```
For the sender (Alice), `initSender` generates its own ephemeral DHs internally — no second argument needed.
