# HASHPIXELS Mining Website — Production

Domain target:
https://www.mine.deadpixelslabs.com/

Contract:
0xB35774851217255a5e1ff08a9752101FCC71fFC8

Chain:
Robinhood Chain mainnet (4663)

Important security behavior:
- No automatic wallet connection.
- No `eth_requestAccounts` on page load.
- No seed phrase/private key input.
- No token approval flow.
- Wallet is requested only when the user explicitly presses CONNECT.
- Claim transaction is requested only after a GPU proof is found and the user presses CLAIM & BURN.

Deploy:
1. Extract this folder.
2. Open PowerShell in the folder.
3. `npx.cmd vercel`
4. Create a NEW Vercel project for the mining website (do not use the asset server project).
5. `npx.cmd vercel --prod`
6. Attach `www.mine.deadpixelslabs.com` to the mining project.

Before public launch:
- Activate mining on the HASHPIXELS contract only when ready.
- Test with a wallet that owns a transferable DEAD PIXELS.
- Confirm WebGPU works in current Chrome/Edge over HTTPS.

Wallet-connect fix:
- Supports OKX direct injection, window.ethereum, multi-provider arrays, and EIP-6963 provider discovery.
- Still NO auto-connect and NO account request on page load.
- CONNECT visibly changes to CONNECTING… and shows a clear error if no provider is available.
- Ethers CDN uses Cloudflare cdnjs with unpkg fallback.

Critical fix:
- Ethers v6 reserves `contract.target` for the contract address.
- The Solidity public getter `target()` is now called via `getFunction("target").staticCall()`.
- This fixes `hashpixelsRead.target is not a function` and restores live chain state / START GPU MINING enablement.
- START remains intentionally disabled until the connected wallet verifies ownership of a DEAD PIXELS token ID.

Mining-loop network stability fix:
- After manual wallet connection, live contract reads use the wallet's injected RPC provider instead of the public browser RPC.
- While hashing, the miner only syncs currentChallenge + target every ~3 seconds instead of doing a full multi-call refresh every ~0.8 seconds.
- Temporary fetch/RPC failures no longer interrupt GPU hashing or spam the console.
- Full dashboard refresh remains slower when idle.

SMOOTH CONTINUOUS MINER
- Triple-buffered WebGPU pipeline (3 independent compute/readback slots).
- CPU readback no longer leaves the GPU waiting between every batch.
- On-chain challenge/target polling runs in a separate background watcher.
- Slow RPC responses never block the hashing loop.
- UI/DOM updates are throttled to ~4 updates/sec instead of every GPU batch.
- Hashrate display uses an EMA for a steadier reading.
- Shorter GPU dispatch targets improve browser/desktop responsiveness.
- Miner automatically switches to a new challenge detected on-chain.
- A locally valid but stale proof automatically restarts mining on the latest challenge.
- Wallet behavior remains manual: no auto-connect, no signatures on load.


RPC TRUTH FIX
- CRITICAL: injected wallet RPC is now signing-only.
- currentChallenge / target / totalMined always come from the official Robinhood RPC.
- Fixes the bug where connecting/starting could rewind the UI to an older totalMined value.
- Monotonic guard rejects any totalMined or L2 block rollback.
- A stale RPC response can no longer rewind the GPU to an old challenge.
- Claim performs a fresh canonical chain-state check before requesting a wallet transaction.
- Stale proofs are discarded instead of allowing repeated CLAIM retries.
- No contract change and no auto-connect behavior.
