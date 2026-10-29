# Olla Finance AZTEC payout audits

Public audit artifacts for Olla Finance (`ollafinance`) provider off-chain delegator reward payouts, operated by Aztlan Labs.

Historical artifacts may retain the former Aztecscan operator name. They are preserved as published; that legacy name does not indicate current explorer ownership or support.

## Operator

- Operator: Olla Finance / Aztlan Labs
- Aztec staking provider id: `4`
- Distribution wallet / Safe: `0x90e7b822a5Ac10edC381aBc03d94b866e4B985A1`
- Reward token: AZTEC `0xA27EC0006e59f245217Ff08CD52A7E8b169E62D2`
- Ethereum mainnet StakingRegistry: `0x042dF8f42790d6943F41C25C2132400fd727f452`
- Current Aztec mainnet v6 rollup: `0xe37a2663cc9c119ba41572c40fab0bbb23a5402f`
- Historical v5 rollup: `0x91ff8bbd8ebb07893010d50a48a1609e5ebd8e34`
- Historical v4 rollup: `0xAe2001f7e21d5EcABf6234E9FDd1E76F50F74962`

## Policy

- Commission: `2500` bps = 25%
- Cadence: weekly
- First off-chain payout started at v4 Aztec epoch `3943`.
- Settlement windows are inclusive and non-overlapping within each rollup. The next run for the same rollup should start at the previous run's `toEpoch + 1`; epoch numbers can restart across upgrades.

## Runs

`runs/` contains epoch-bound payout audits and payout Safe imports. Check each audit's `rollup` address as well as its epoch window. A published Safe import is not evidence that its transactions have executed.

Recent audits:

| Rollup | Window | Artifacts |
|---|---|---|
| v6 | epochs `603-717` | [`audit`](./runs/epoch-603-717-mvtdyuhg-247692.json), [`payout Safe import`](./runs/epoch-603-717-mvtdyuhg-247692.safe.json) |
| v5 (final) | epochs `4237-4425` | [`audit`](./runs/epoch-4237-4425-mvrt4qy2-3f1729.json) |

Historical first payout:

| Window | Run id | Delegator payout | Operator commission | Artifacts |
|---|---:|---:|---:|---|
| epochs `3943-4104` | `mqozqmiy-ed7589` | `34,912.5 AZTEC` | `11,637.5 AZTEC` | [`audit`](./runs/epoch-3943-4104-mqozqmiy-ed7589.json), [`safe`](./runs/epoch-3943-4104-mqozqmiy-ed7589.safe.json) |

## Reusable reward claims

`claims/` contains reusable, rollup-specific Safe imports, separate from epoch payout runs:

- [v6 sequencer reward claim](./claims/v6-0xe37a2663cc9c119ba41572c40fab0bbb23a5402f.safe.json)

The v6 claim calls `claimSequencerRewards(0x90e7b822a5Ac10edC381aBc03d94b866e4B985A1)` on the v6 rollup on Ethereum mainnet (chain ID `1`), with zero ETH value. It sweeps the Safe's available sequencer rewards, not a specific epoch window, and can be reused before a payout batch. It does not transfer delegator payouts. Verify the target, recipient, and available rewards before execution.

## Verification

The audit JSON records the settlement window, reward parameters, attributed checkpoints, per-delegator transfer amounts, and encoded transaction calldata.

To reproduce a run, use the same payout runner version and configuration as that run, plus your own archival Ethereum mainnet RPC URL. `config.public.yaml` currently records the historical v5 rollup; it is not a v6 configuration. For v6, use the rollup and reward parameters recorded in the corresponding audit.

For the historical first v4 run (the source URL below retains its historical organization name), also set `rollupAddress` to the v4 address above:

```bash
git clone https://github.com/aztec-scan/aztec-staking-payout.git
cd aztec-staking-payout
git checkout b2bc318
cp ../aztec-payout-audit/config.public.yaml ./config.yaml
# edit config.yaml and add your own archival rpcUrl
npm install
npm run settle -- --config ./config.yaml --from-epoch 3943 --to-epoch 4104 --dry-run
```

The generated payout amounts should match the audit file for the same epoch window.
