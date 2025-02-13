# Guide for Validator CLI

## Create Wallet
To create a new wallet, use the command below. Don’t forget to save the mnemonic:
```
fuelsequencerd keys add wallet
```

To recover your wallet using a seed phrase:
```
fuelsequencerd keys add wallet --recover
```

Show your wallet list:
```
fuelsequencerd keys list
```

## Save Wallet Info
Add wallet and validator address into variables:
```
export SELFCHAIN_WALLET_ADDRESS=$(fuelsequencerd keys show wallet -a)
export SELFCHAIN_VALOPER_ADDRESS=$(fuelsequencerd keys show wallet --bech val -a)
echo 'export SELFCHAIN_WALLET_ADDRESS='${SELFCHAIN_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SELFCHAIN_VALOPER_ADDRESS='${SELFCHAIN_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Create Validator

Check your wallet balance:
```
fuelsequencerd query bank balances $SELFCHAIN_WALLET_ADDRESS
```

To create your validator, run the command below:
```
fuelsequencerd tx staking create-validator \
  --amount 1000000fuel \
  --from=wallet \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey $(fuelsequencerd tendermint show-validator) \
  --moniker=YourNodeMoniker \
  --chain-id=seq-mainnet-1 \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.005fuel
```

## Staking, Delegation, and Rewards
Delegate stake:
```
fuelsequencerd tx staking delegate $SELFCHAIN_VALOPER_ADDRESS 1000000fuel --from=wallet --chain-id=seq-mainnet-1 --gas=auto
```

Redelegate stake:
```
fuelsequencerd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000fuel --from=wallet --chain-id=seq-mainnet-1 --gas=auto
```

Withdraw all rewards:
```
fuelsequencerd tx distribution withdraw-all-rewards --from=wallet --chain-id=seq-mainnet-1 --gas=auto
```

Withdraw rewards with commission:
```
fuelsequencerd tx distribution withdraw-rewards $SELFCHAIN_VALOPER_ADDRESS --from=wallet --commission --chain-id=seq-mainnet-1
```

## Validator Management
Edit validator:
```
fuelsequencerd tx staking edit-validator \
  --moniker=YourNodeMoniker \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=seq-mainnet-1 \
  --from=wallet
```

Unjail validator:
```
fuelsequencerd tx slashing unjail --from=wallet --chain-id=seq-mainnet-1 --gas=auto
```
