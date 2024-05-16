# Overview

In this step, operators are required to sign the grants transction.

Five months ago, another transaction was signed in order to grant some x/authz authorizations to the coordinator account so that it can perform some operations on behalf of the multisig account. However, that transaction did not contain all the messages that we actually need. 
For this reason, the transaction that we have to generate contains the following messages:

1. Three `MsgGrant` messages that allow the coordinator account to stake, redelegate and undelegate to and from any validator without any limitation on behalf of the multisig account.
This message will automatically revoke the AUTHORIZATION_TYPE_DELEGATE, the AUTHORIZATION_TYPE_UNDELEGATE and the AUTHORIZATION_TYPE_REDELEGATE stake authorizations currently present.

2. A `MsgRevokeAllowance` message to revoke the current x/feegrant allowance present. If we want to be able to pay for fees using the multisig wallet, we need to have a fee grant for the `MsgExec` message type instead. This is why we need to remove the current fee grant to be able to re-create it with `MsgExec`.

3. A `MsgGrantAllowance` message that allows the coordinator to pay for fees using the multisig account balance. The feegrant is limited to the `/cosmos.authz.v1beta1.MsgExec` and the `/ibc.applications.transfer.v1.MsgTransfer` messages types.

## Multisig Accounts

The following multisig accounts are involved for this signing process.

| Name | Address |
|---|---|
| `Staker`            |`celestia1vxzram63f7mvseufc83fs0gnt5383lvrle3qpt` |
| `Staker Controller` |`celestia16g5l6n9kg6879z695g6qjh70qv6wzqg640z9pn` |
| `Grantee`           |`celestia1fl85qh9pw4ju48zy2eqr8h0h6d8hwgpq5880wd` |

## Signing signatures

For your convenience, the following JSON files are prepared.

- `/2024-05-16/signatures/unsigned_tx.json`

This file includes the following grants:
- [MsgGrant/MsgRevokeAllowance](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/feegrant/v1beta1/tx.proto#L20)
    - This grant is needed for the `Staker Controller` to revoke existing fee allowance so that it can `MsgRevokeAllowance` and `MsgGrantAllowance` with the new options.
- [MsgRevokeAllowance](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/feegrant/v1beta1/tx.proto#L20)
    - This message is needed for the `Grantee` to use transaction fees from the `Staker` multisig account.
- [MsgGrant/MsgDelegate](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/staking/v1beta1/tx.proto#L26)
    - This grant is needed for the `Grantee` to delegate on behalf of the `Staker` multisig. It is important to note that the grant is setup with an allow list of validators selected by the MilkyWay protocol. With this constraint, the `Grantee` is only permitted to delegate among the validators listed in the allow list.
- [MsgGrant/MsgBeginRedelegate](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/staking/v1beta1/tx.proto#L30)
    - This grant is needed for the `Grantee` to redelegate delegations from the `Staker` multisig account in case the validator set is changed. It is important to note that the grant is setup with an allow list of validators selected by the MilkyWay protocol. With this constraint, the `Grantee` is only permitted to redelegate among the validators listed in the allow list.
- [MsgGrant/MsgUndelegate](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/staking/v1beta1/tx.proto#L34)
    - This grant is needed for the `Grantee` to undelegate on behalf of the `Staker` multisig. It is important to note that the grant is set up with an allow list of validators selected by the MilkyWay protocol. With this constraint, the `Grantee` is only permitted to undelegate among the validators listed in the allow list.
- [MsgGrantAllowance](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/feegrant/v1beta1/tx.proto#L16)
    - This grant is needed for the `Grantee` to pay for fees using the multisig account balance

```bash
# Import your `Staker Controller` wallet
# Change index to the one that you have used when setting the account
celestia-appd keys add op-staker-controller --ledger --index 1

# We use the following public RPC endpoint to get account number.
# In case it is not responsive, use any alternatives in the following link
# https://docs.celestia.org/nodes/mainnet
NODE="https://celestia-rpc.mesa.newmetric.xyz:443"
CONTROLLER_ADDR="celestia16g5l6n9kg6879z695g6qjh70qv6wzqg640z9pn"

# Sign the unsigned transaction
# Make sure you change the `OPERATOR_NAME`
celestia-appd tx sign unsigned_tx.json \
--chain-id celestia \
--from op-staker-controller \
--multisig $CONTROLLER_ADDR \
--ledger \
--node $NODE \
--output-document=signature_{OPERATOR_NAME}.json
```

## Submitting your signature

1. Fork this repository.

2. Copy the `OPERATOR_NAME.txt` file and change the file name to your operator name.

3. Input your signatures under the `2025-05-16/signatures` directory

4. Create a pull request