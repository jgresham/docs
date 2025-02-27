---
title: Contract Wallet Stake Migration Guide
sidebar_position: 11
---

This guide provides instructions for Livepeer participants using a contract wallet (i.e., multisig) to stake on L1 (Ethereum Mainnet), to migrate this stake to L2 (Arbitrum Mainnet) as per the [Livepeer Confluence upgrade](https://medium.com/livepeer-blog/the-confluence-upgrade-is-live-3b6b342ea71e). 

This guide is designed to be used in conjunction with the [Livepeer Explorer L2 Contract Wallet Migration Tools](https://explorer.livepeer.org/migrate) to support migrating to Arbitrum: 

- Orchestrators - Migrate stake and fees
- Broadcasters - Migrate your deposit and reserve
- Delegators - Migrate undelegated stake 

The migration process will transfer ownership of funds (i.e., stake) owned by your L1 address to the L2 address.

## Create an L2 Address

**Before moving your stake, you MUST confirm that you have a valid L2 address that can own funds migrated from L1. Otherwise, you could lose access to your L1 assets after the migration if you use the L1 address of your contract wallet as the L2 address**

The L2 address can be that of a contract wallet (i.e. [Gnosis Safe](https://gnosis-safe.io/)) deployed on L2, or an EOA (an externally owned account managed by a hardware wallet (e.g., Ledger)), or a browser wallet (e.g., Metamask). 

**If you would like to use a contract wallet on L2 you MUST make sure that the contract wallet is deployed on L2 and that you have control of it before proceeding.**

> **Note:** If you are unsure about how to ensure you have a valid L2 address, please **reach out to the core team on Discord** in the [#layer2-confluence-support channel](https://discord.gg/5eQ3YfK2a8).

## Migrate Stake

1. Use the [Contract Wallet Tool](https://explorer.livepeer.org/migrate/delegator/contract-wallet-tool) to generate the required parameters for the migration transactions. These parameters will be required in upcoming steps: 

1. Once you enter the L1 address, and an L2 address (the address you created in the prior step), the tool will display the following:

- The `migrateDelegator` transaction parameters if the L1 address has stake to migrate
- The `migrateUnbondingLocks` transaction parameters if the L1 address has unbonding locks (i.e., undelegated stake) to migrate

### Submit a Stake Migration Transaction

If your L1 address has stake to migrate: 

- Submit the stake migration transaction from your contract wallet, a function call to the L1Migrator on **Ethereum Mainnet**.

- The contract address is `0x21146B872D3A95d2cF9afeD03eE5a783DaE9A89A`
- Copy the [contract ABI](https://etherscan.io/address/0x21146B872D3A95d2cF9afeD03eE5a783DaE9A89A#code)
- The function name is `migrateDelegator` with the following parameters:
    - `address _l1Addr`: the address of your L1 contract wallet that has stake to migrate
    - `address _l2Addr`: the address on L2 that will receive migrated stake (the L2 address you created prior).
    - `bytes _sig`: this parameter can be left blank
    - `uint256 _maxGas`: the `maxGas` printed by the command-line tool
    - `uint256 _gasPriceBid`: the `gasPriceBid` printed by the command-line tool
    - `uint256 _maxSubmissionCost`: the `maxSubmissionCost`  printed by the command-line tool
- The ETH value to include with the function call should be the `value` printed by the command-line tool. The `value` printed by the command-line tool is **denominated in Wei**, so make sure to convert it into the units (i.e., Ether) required by the tool you are using to submit the transaction.

### Submit an Unbonding Lock Migration Transaction

If your L1 address has unbonding locks to migrate, submit an unbonding locks migration transaction from your contract wallet; a function call to the L1Migrator on **Ethereum Mainnet**.

- The contract address is `0x21146B872D3A95d2cF9afeD03eE5a783DaE9A89A`
- Copy the [contract ABI](https://etherscan.io/address/0x21146B872D3A95d2cF9afeD03eE5a783DaE9A89A#code)
- The function name is `migrateUnbondingLocks` with the following parameters:
    - `address _l1Addr`: the address of your L1 contract wallet that has unbonding locks to migrate
    - `address _l2Addr`: the address on L2 that will receive the total stake of migrated unbonding locks (the L2 address you created prior).
    - `uint256[] _unbondingLockIds`: The array of IDs for unbonding locks that will be migrated
    - `bytes _sig`: This parameter can be ignored and left blank
    - `uint256 _maxGas`: This should be the `maxGas` printed by the command-line tool
    - `uint256 _gasPriceBid`: This should be the `gasPriceBid` printed by the command-line tool
    - `uint256 _maxSubmissionCost`: This should be the `maxSubmissionCost`  printed by the command-line tool
- The ETH value to include with the function call should be the `value` printed by the command-line tool. The `value` printed by the command-line tool is **denominated in Wei**, so make sure to convert it into the units (i.e., Ether) required by the tool you are using to submit the transaction.

### Waiting For Transaction Finalization on L2

After a migration transaction is submitted by the contract wallet and confirmed on L1, it takes about ~10 minutes to finalize on L2. You can use a command-line tool to confirm the transaction is finalized on L2:

```bash
# Clone the repository
git clone https://github.com/livepeer/arbitrum-lpt-bridge
# Navigate into the repository
cd arbitrum-lpt-bridge
# Install dependencies
yarn
# Set environment variables
export MAINNET_URL=<ETHEREUM_MAINNET_RPC_URL>
export ARB_MAINNET_URL=<ARBITRUM_MAINNET_RPC_URL>
# Run command
npx hardhat wait-to-relay-tx-to-l2 <L1_TX_HASH>
```

- `<L1_TX_HASH>` is the hash of the L1 transaction submitted by your contract wallet. 

    The command will notify you when the transaction is finalized on L2. 
    You should then be able to navigate to the [Livepeer Explorer](https://explorer.livepeer.org/) to find the L2 address you specified and view the migrated stake owned by that address.
