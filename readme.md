# Creating an NFT

Following steps allow to create an NFT using a minting policy of type Time-locked in testnet.

## Location of files
- Keys for payment address are located on /key1
- Keys for policy are located on main folder

## Steps
1. Upload to ipfs
```
curl "https://ipfs.blockfrost.io/api/v0/ipfs/add" \
  -X POST \
  -H "project_id: ipfsWRpGJnKjMALDhlHZc13mPzpK11eeqRom" \
  -F "file=@./nano.png"
```
2. Pin it
```
curl "https://ipfs.blockfrost.io/api/v0/ipfs/pin/add/QmWZ5BxqaE4XV6Z5HavTJHdazL2tYdF6MraFGJAh2ZY6r6" \
  -X POST \
  -H "project_id: ipfsWRpGJnKjMALDhlHZc13mPzpK11eeqRom"
```
3. Create a policy key
```
cardano-cli address key-gen \
    --verification-key-file policy.vkey \
    --signing-key-file policy.skey
```
4. Hash of the verification key file
```
cardano-cli address key-hash --payment-verification-key-file policy.vkey
```

> Hash: e06e4bd41e01aa5449a9c18ea444d21bdb12f1a1a90ab087805ebc76

5. Find the slot in the testnet
```
cardano-cli query tip --testnet-magic 1097911063
```

> or using https://explorer.cardano-testnet.iohkdev.io/

6. Build the policy.json: https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/simple-scripts.md

7. Hash of the policy script
```
cardano-cli transaction policyid --script-file policy.json > policy.id
```

> Policy id: 8f7633ebcf477753565e87f69e1d9c791bc39adccd10390f33720ab3

8. Define metadata.json including the policy id
9. Get protocol parameters
```
cardano-cli query protocol-parameters --testnet-magic 1097911063 --out-file protocol-params.json
```

10. Query UTxO
```
cardano-cli query utxo --address $(cat key1/payment.addr) --testnet-magic 1097911063
```

11. Build transaction
```
cardano-cli transaction build-raw \
  --fee 0 \
  --tx-in 722f716a16286dfaf21925e7f7013f2e8248223ed8f04b455c9a745751a4c6d9#0 \
  --tx-out $(cat key1/payment.addr)+200000000+"1 $(cat policy.id).nano0001" \
  --mint="1 $(cat policy.id).nano0001" \
  --minting-script-file policy.json \
  --metadata-json-file metadata.json \
  --invalid-hereafter=40590754 \
  --out-file transaction.raw
```

12. Calculate min fee
```
cardano-cli transaction calculate-min-fee \
  --tx-body-file transaction.raw \
  --tx-in-count 1 \
  --tx-out-count 1 \
  --witness-count 2 \
  --testnet-magic 1097911063 \
  --protocol-params-file protocol-params.json
  ```

> Min fee: 194541 Lovelace
> Change: 200000000 - 194541 = 199805459

13. Build final transaction using fees and change
```
cardano-cli transaction build-raw \
  --fee 194541 \
  --tx-in 722f716a16286dfaf21925e7f7013f2e8248223ed8f04b455c9a745751a4c6d9#0 \
  --tx-out $(cat key1/payment.addr)+199805459+"1 $(cat policy.id).nano0001" \
  --mint="1 $(cat policy.id).nano0001" \
  --minting-script-file policy.json \
  --metadata-json-file metadata.json \
  --invalid-hereafter=40593754 \
  --out-file transaction.raw
```

14. Sign the transaction
```
cardano-cli transaction sign \
  --signing-key-file key1/payment.skey \
  --signing-key-file policy.skey \
  --testnet-magic 1097911063 \
  --tx-body-file transaction.raw \
  --out-file transaction.signed
```

15. Submit the transaction
```
cardano-cli transaction submit \
  --tx-file transaction.signed \
  --testnet-magic 1097911063
```

16. Confirm transaction from testnet.cardanoscan.io
> https://testnet.cardanoscan.io/tokenPolicy/8f7633ebcf477753565e87f69e1d9c791bc39adccd10390f33720ab3

## Transfer the NFT

17. Build the transaction
```
cardano-cli transaction build \
  --alonzo-era \
  --tx-in c2f623a2c3a0d05a7f94a8a2db6921f27cf281bc8f489b7861c7b9bbe5a8059a#0 \
  --tx-out addr_test1qquw7k22ae7xf772l4ljuvp7ncsq4uzsl2zlvtqlr5kf9lclzytcyjujtzmedxzjgq92kwg48my4dsdnzcmdj6eh5sxqy4j3up+5000000+"1 $(cat policy.id).nano0001" \
  --change-address $(cat key1/payment.addr) \
  --testnet-magic 1097911063 \
  --out-file transfer.raw
```

18. Sign the transaction
```
cardano-cli transaction sign \
  --signing-key-file key1/payment.skey \
  --testnet-magic 1097911063 \
  --tx-body-file transfer.raw \
  --out-file transfer.signed
```

19. Submit the transaction
```
cardano-cli transaction submit \
  --testnet-magic 1097911063 \
  --tx-file transfer.signed
```