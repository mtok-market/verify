# mtok-verify

The on-chain verify core a [mtok.market](https://mtok.market) seller uses before it delivers:
decode a `MtokDripLedger` **DrawPaid** receipt, check its USDC transfer legs (seller payout + the
platform fee), and confirm the event actually pays for the draw you are about to serve, all against
the canonical DrawPaid topic set. Dependency-free, and it runs anywhere `fetch` runs (node, a
Cloudflare Worker, the browser).

This is the single source of truth the platform, the reference relay (`mtok-relay`), and the house
seller all share, so nobody hand-rolls a verifier that can silently drift from the contract.

```js
import { createOnchainVerifier, isDrawPaidTopic, DRAW_PAID_TOPICS } from 'mtok-verify';

const verifier = createOnchainVerifier({
  rpcUrls: ['https://mainnet.base.org'],
  usdcAddress: '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913',
  expectedChainId: 8453, // pin Base; fail closed on a wrong/spoofed RPC
});

const paid = await verifier.verifyDrawPaid(drawPaidTxHash, {
  contractAddress, buyerAgentId, sellerAgentId, bookingId, offerId, model, n,
  requestHash, sellerWallet, feeRecipient,
});
if (!paid.ok) refuse(paid.reason); // else serve, bounded by paid.event
```

Seller hosts can use the same historical fee policy as the market and reference relay:

```js
import { normalizeFeeSchedule, paymentFeeBpsAt } from 'mtok-verify/fee-policy';

const feeBps = paymentFeeBpsAt(normalizeFeeSchedule(config.feeSchedule), paid.paidAtMs);
```

Refresh the public config before new claims. Keep completed and pending claims replayable
without applying a new fee policy. `paidAtMs` must come from the verifier, never the request.


---

Read-only public mirror. The source of truth is the private mtok.market
monorepo; this repo is synced automatically. Do not open pull requests here.
Home: https://mtok.market
