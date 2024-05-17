# Overview

Five months ago, another transaction was signed in order to grant some x/authz authorizations to the coordinator account so that it can perform some operations on behalf of the multisig account. However, that transaction did not contain all the messages that we actually needed for the coordinator to run properly. For this reason, we now need to perform another transaction that contains the following messages:

1. Three `MsgGrant` messages that allow the coordinator account to stake, redelegate and undelegate to and from any validator without any limitation, on behalf of the multisig account.
This message will automatically revoke the `AUTHORIZATION_TYPE_DELEGATE`, the `AUTHORIZATION_TYPE_UNDELEGATE` and the `AUTHORIZATION_TYPE_REDELEGATE` stake authorizations currently present.

3. A `MsgRevokeAllowance` message to revoke the current x/feegrant allowance present.  
If we want to be able to pay for fees using the multisig wallet, we need to have a fee grant for the `MsgExec` message type. This is why we need to remove the current fee grant to be able to re-create it including such message type.

5. A `MsgGrantAllowance` message that allows the coordinator to pay for fees using the multisig account balance.
The feegrant is limited to the `/cosmos.authz.v1beta1.MsgExec` and the `/ibc.applications.transfer.v1.MsgTransfer` messages types.

## Multisig Accounts

The following multisig accounts are involved for this signing process.

| Name | Address |
|---|---|
| `Staker`            |`celestia1vxzram63f7mvseufc83fs0gnt5383lvrle3qpt` |
| `Staker Controller` |`celestia16g5l6n9kg6879z695g6qjh70qv6wzqg640z9pn` |
| `Grantee`           |`celestia1fl85qh9pw4ju48zy2eqr8h0h6d8hwgpq5880wd` |

## Transaction contents

For your convenience, the following JSON file has been prepared:

- [/2024-05-16/signatures/unsigned_tx.json](https://github.com/milkyway-labs/operators/blob/main/2024-05-16/signatures/unsigned_tx.json)

This file includes the following grants:
- [MsgGrant/MsgRevokeAllowance](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/feegrant/v1beta1/tx.proto#L20)
    - This grant is needed for the `Staker Controller` to be able to revoke the existing fee allowances so that it can properly execute  `MsgRevokeAllowance` and `MsgGrantAllowance` with the new options.
- [MsgRevokeAllowance](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/feegrant/v1beta1/tx.proto#L20)
    - This message is needed for the `Grantee` to be able to pay for transaction fees using the balance of the `Staker` multisig account.
- [MsgGrant/MsgDelegate](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/staking/v1beta1/tx.proto#L26)
    - This grant is needed for the `Grantee` to delegate on behalf of the `Staker` multisig to various validators.
- [MsgGrant/MsgBeginRedelegate](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/staking/v1beta1/tx.proto#L30)
    - This grant is needed for the `Grantee` to redelegate delegations from the `Staker` multisig account in case the validator set is changed.
- [MsgGrant/MsgUndelegate](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/staking/v1beta1/tx.proto#L34)
    - This grant is needed for the `Grantee` to undelegate on behalf of the `Staker` multisig.
- [MsgGrantAllowance](https://github.com/cosmos/cosmos-sdk/blob/v0.46.14/proto/cosmos/feegrant/v1beta1/tx.proto#L16)
    - This grant is needed for the `Grantee` to pay for fees using the multisig account balance.

### About MsgDelegate, MsgBeginRedelegate and MsgUndelegate grants

When you read the transaction, you'll notice that the new `MsgDelegate`, `MsgBeginRedelegate`, and `MsgUndelegate` grants don't specify any particular list of allowed validators. This is because if we included such a list, we couldn't implement the dynamic addition and removal of validators. So, we require a generic grant instead, without limitations on the list of validators for delegation/redelegation/undelegation.

This change doesn't pose any security risk: since the `Grantee` can only perform staking operations, the worst-case scenario is mismanagement of delegations. However, this can easily be solved by signing and broadcasting a transaction to adjust the delegation distribution and revoke granted permissions using the multisig wallet.

## Creating the signature

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

2. Sign tx with unsigned_tx.json file

3. Input your signatures(signature_{OPERATOR_NAME}.json) under the `2025-05-16/signatures` directory

4. Create a pull request
