# Guide for Validator CLI

## Create Wallet
To create a new wallet, use the command below. Don’t forget to save the mnemonic:
```
uniond keys add wallet
```

To recover your wallet using a seed phrase:
```
uniond keys add wallet --recover
```

Show your wallet list:
```
uniond keys list
```

## Save Wallet Info
Add wallet and validator address into variables:
```
export UNION_WALLET_ADDRESS=$(uniond keys show wallet -a)
export UNION_VALOPER_ADDRESS=$(uniond keys show wallet --bech val -a)
echo 'export UNION_WALLET_ADDRESS='${UNION_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export UNION_VALOPER_ADDRESS='${UNION_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Create Validator

Check your wallet balance:
```
uniond query bank balances $UNION_WALLET_ADDRESS
```
To create your validator, run the command below:
```
uniond comet show-validator
```
The output will be similar to this (with a different key):
```
{"@type":"/cosmos.crypto.bn254.PubKey","key":"xxxxxxxxxxxxxxxxxx="}
```
Create a Proof of Possession
```
export POSSESSION_PROOF=$(uniond prove-possession $(jq -r 'priv_key.value' ~/.union/config/priv_validator_key.json))
```
Then, create a file named validator.json with the following content:
```
{
    "pubkey": {"@type":"/cosmos.crypto.bn254.PubKey","key":"xxxxxxxxxxxxxxxxxxxxx"},
    "amount": "1000000muno",
    "moniker": "your-node-moniker",
    "identity": "testnet validator",
    "website": "optional website for your validator",
    "security": "optional security contact for your validator",
    "details": "optional details for your validator",
    "commission-rate": "0.1",
    "commission-max-rate": "0.2",
    "commission-max-change-rate": "0.01",
    "min-self-delegation": "1"
}
```
Finally, we’re ready to submit the transaction to create the validator:
```
uniond union-staking create-union-validator validator.json    --from wallet   --chain-id union-testnet-9
```
## Staking, Delegation, and Rewards
Delegate stake:
```
uniond tx staking delegate $UNION_VALOPER_ADDRESS 1000000muno --from=wallet --chain-id=union-testnet-9 --gas=auto
```

Redelegate stake:
```
uniond tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000muno --from=wallet --chain-id=union-testnet-9 --gas=auto
```

Withdraw all rewards:
```
uniond tx distribution withdraw-all-rewards --from=wallet --chain-id=union-testnet-9 --gas=auto
```

Withdraw rewards with commission:
```
uniond tx distribution withdraw-rewards $UNION_VALOPER_ADDRESS --from=wallet --commission --chain-id=union-testnet-9
```

## Validator Management
Edit validator:
```
uniond tx staking edit-validator \
  --moniker=<Your_Nodename_Moniker> \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=union-testnet-9 \
  --from=wallet
```

Unjail validator:
```
uniond tx slashing unjail --from=wallet --chain-id=union-testnet-9 --gas=auto
```


