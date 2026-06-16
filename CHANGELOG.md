# Changelog: `dapp-connect` review feedback

> This log documents the non-blocking reviewer feedback applied to the `midnight-dapp-connect` standalone repo. Each item shows the original issue, the exact change made in `src/pages/Transfer.tsx` and/or `tutorial.md`, and the result.

---

## 1. Use `guaranteedUnshieldedOffer` for plain transfers

**Feedback:** `Transfer.tsx` assigned the offer to `(intent as any).fallibleUnshieldedOffer`. A plain unshielded transfer belongs in segment 0 (`guaranteedUnshieldedOffer`); the fallible segment is for offers that may fail alongside a guaranteed section.

### `src/pages/Transfer.tsx`

**Before:**

```typescript
const intent = Intent.new(new Date(Date.now() + 30 * 60 * 1000));
(intent as any).fallibleUnshieldedOffer = unshieldedOffer;

const unsealedTx = Transaction.fromParts('preprod', undefined, undefined, intent as any);
```

**After:**

```typescript
const intent = Intent.new(new Date(Date.now() + 30 * 60 * 1000));
intent.guaranteedUnshieldedOffer = unshieldedOffer;

const unsealedTx = Transaction.fromParts('preprod', undefined, undefined, intent);
```

### `tutorial.md`

Added an explanation in the "Why each step matters" subsection:

> **`guaranteedUnshieldedOffer`:** A plain unshielded transfer belongs in the *guaranteed* segment of the intent (segment 0). The `fallibleUnshieldedOffer` segment is for offers that may fail alongside a guaranteed section. Using `guaranteedUnshieldedOffer` matches the reference implementations and removes the need for an `as any` cast.

**Result:** The transfer now uses the correct intent segment, removes the `as any` workaround, and the tutorial explains why.

---

## 2. Recipient field starts empty

**Feedback:** The transfer page prefilled the recipient with the connected wallet's own address, making the default action a self-send. Starting empty makes for a clearer demo.

### `src/pages/Transfer.tsx`

**Before:**

```typescript
useEffect(() => {
  if (isConnected) {
    loadWalletState();
    if (addresses?.unshieldedAddress && !recipient) {
      setRecipient(addresses.unshieldedAddress);
    }
  }
}, [isConnected, addresses?.unshieldedAddress]);
```

**After:**

```typescript
useEffect(() => {
  if (isConnected) {
    loadWalletState();
  }
}, [isConnected]);
```

The **Use my address (self-transfer)** button is kept, so users can still safely test a self-send with one click.

### `tutorial.md`

Added context before the `Transfer.tsx` snippet:

> The recipient field starts empty; a **Use my address (self-transfer)** button prefills it with the connected wallet's unshielded address so you can safely test with a self-send.

**Result:** The default is no longer a hidden self-send. Users must explicitly choose a recipient, while a one-click self-transfer remains available.

---

## 3. Remove "Browser vs CLI" callout from `Transfer.tsx`

**Feedback:** The comparison block rendered at the bottom of the transfer page reads better as part of the written tutorial than in the running DApp.

### `src/pages/Transfer.tsx`

**Before:** The transfer page ended with a styled "Browser vs CLI" comparison card.

**After:** The comparison card is removed. The browser-vs-CLI comparison remains in the tutorial's `## Architecture: browser vs CLI` section.

**Result:** The transfer UI is focused on the transfer flow. The conceptual comparison stays in the tutorial where it belongs.

---

## 4. Trim CLI depth in the tutorial

**Feedback:** The full CLI build-out (key derivation, `WalletFacade` init, dust sync, runnable transfer script) goes beyond what a browser connector tutorial needs. The conceptual comparison already satisfies the `balanceUnboundTransaction` vs browser-flow deliverable.

### `tutorial.md`

**Before:** The CLI section included:
- Key derivation with BIP-39 mnemonic
- Full `WalletFacade` initialisation with all v3 config fields
- Dust sync handling with 2-hour timeout
- `SIGINT`/`SIGTERM` state persistence
- CLI transfer snippet
- Run instructions

**After:** The section keeps the high-level comparison and the minimal `transferTransaction` + `signRecipe` snippet, then points to the full scripts for details:

> The `ctx` object is built from a 24-word mnemonic via `restoreWalletState()`; the full key derivation, `WalletFacade` initialisation, dust sync handling, and state persistence live in [`src/lib/transaction-cli.ts`](https://github.com/0xfdbu/midnight-dapp-connect/blob/main/src/lib/transaction-cli.ts) and [`scripts/test-v3-sync-and-transfer.ts`](https://github.com/0xfdbu/midnight-dapp-connect/blob/main/scripts/test-v3-sync-and-transfer.ts). Run the script with: ...

The CLI code files in `src/lib/transaction-cli.ts` and `scripts/test-v3-sync-and-transfer.ts` are left untouched.

**Result:** The tutorial stays focused on the browser connector flow while still acknowledging the CLI alternative and linking to the runnable implementation.

---

## 5. Clarify wording around `balanceUnsealedTransaction` and `result.tx`

**Feedback:**
- The tutorial says the wallet "balances, signs, and pays fees" via `balanceUnsealedTransaction` but doesn't say what "unsealed" means.
- `result.tx.slice(0, 64)` is shown as a txId, but it's the first 64 characters of the serialised balanced transaction, not a transaction hash.

### `tutorial.md`

**Before:**

> **`tx.prove()`:** `balanceUnsealedTransaction` expects a transaction with the `Proof` marker. Without `prove()`, the transaction serialises with `proof-preimage` (`PreProof` state) and the wallet rejects it with: ...

> // 5. Submit
> await connectedApi.submitTransaction(result.tx);
>
> setTxId(result.tx.slice(0, 64));

**After:**

> **`tx.prove()` and "unsealed":** `balanceUnsealedTransaction` expects a transaction with the `Proof` marker. An *unsealed* transaction is already proven (`tx.prove()`) but has not yet been cryptographically bound. The wallet applies that binding while it balances the transaction and pays fees. Without `prove()`, the transaction serialises with `proof-preimage` (`PreProof` state) and the wallet rejects it with: ...

> **Transaction identifier display:** `result.tx` is the serialised balanced transaction, not its hash. The code displays `result.tx.slice(0, 64)` — the first 64 hex characters of that serialised transaction — as a concise success indicator. If you need the real transaction hash, compute it from the submitted transaction bytes.

**Result:** Readers now understand what "unsealed" means and know that the displayed value is a transaction prefix, not a hash.

---

## 6. Style guide cleanup

Part of the required feedback was to align the repo with the Tech Content Style Guide (`DApp` capitalization, British spelling).

### `README.md`

- Changed the project structure tree root from `Dapp-connect/` to `midnight-dapp-connect/` to follow the repo name and avoid the non-standard `Dapp` casing.

### `tutorial.md`

- Added H1 title using `DApp` capitalization: `# [Tutorial] DApp Connector API: Connecting a browser DApp to Midnight wallets`.
- British spelling pass: `initialise/initialisation`, `serialise/serialised`.

**Result:** No prose occurrences of `Dapp` or lowercase standalone `dapp` remain outside repo/package names and code identifiers.

---

## Files changed

- `src/pages/Transfer.tsx`
- `tutorial.md`
- `README.md`
- `CHANGELOG.md` (this file)
