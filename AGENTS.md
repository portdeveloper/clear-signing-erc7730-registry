# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, Windsurf, and others) working in this repository. This repo is the ERC-7730 clear-signing registry: JSON descriptors that tell wallets how to render a contract's calls and signed messages in plain language instead of raw hex.

If you are in Cursor, the `.cursor/rules/` directory has detailed, tested rules for these tasks and is the canonical source for exact commands. This file is the vendor-neutral summary so any agent can follow the same workflow.

## Your job

Help a contributor produce a correct descriptor for their contract and open a PR. Work inside a clone of this repo.

## Workflow

1. Check for an existing descriptor first. Search `registry/` for the contract address and the protocol name. If a descriptor already exists and only the chain is missing, add `{ "chainId": <id>, "address": "<addr>" }` to its `context.contract.deployments` and stop. This is the common case and the smallest correct change.

2. Generate, only if none exists. Use the in-repo generator `node tools/scripts/generate-7730.js` (see `.cursor/rules/generate-7730.mdc` for the exact flags for address, abi, or source, and the LLM backend). It downloads the ABI and source on-chain and emits a `calldata-*.json` or `eip712-*.json`.

3. Verify the address. Confirm there is bytecode at the address on the target chain. For EIP-712 contracts, match the live `DOMAIN_SEPARATOR()` against the value computed for the domain. Do not write down an address you have not confirmed.

4. Write readable intents and labels. The `intent` is the action in plain language and should be 30 characters or fewer, because Ledger devices truncate longer text. Label every field a user should see, like the token, the amount, the spender, the recipient. Use formats such as `tokenAmount` and `addressName` so values render properly, and fall back to `raw` only when nothing else fits. Base labels on the real contract semantics, not on guesses.

5. Validate and test. Lint with the repo tooling (see `.cursor/rules/lint-erc7730.mdc`) and generate tests (`.cursor/rules/generate-tests.mdc`). Nested or arbitrary calldata such as multicall, batch, and permit-with-data cannot be statically decoded. Cover what decodes cleanly and state which functions you left out.

6. Open the PR from an account tied to the contract owner. Maintainers may ask for proof of ownership. The file belongs at `registry/<owner>/<calldata|eip712>-<Name>.json`.

## Notes

- Prefer the smallest correct change. Adding a chain to an existing descriptor beats a new file.
- Do not invent addresses, ABIs, or field meanings. Read the source or ask.
