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

## Creating Collateral

1. Build transaction without fees
```
cardano-cli transaction build-raw \
    --fee 0 \
    --tx-in 09a3d2115eaf7a6bb44ca5adad62170ef737a032728c44b9e219f260085b4db8#0 \
    --tx-out addr_test1qzr8dxy9wy4lyjsrmhufkrcfhdxy27k329z66xs6xqjartzrym79ewvl0rem9r0wk8dtry43hj4nt0ghw09n60v40k3srv5uq3+144783050+"1 a90f59e9ca4020571b2f5dd6188a5a6e5a29a0e787bbbbd19b75deac.nano0002"+"1 c2b777534032e46e86b3c6b198edd8bef5a19026df95d104f44b5415.jorge" \
    --tx-out addr_test1qzr8dxy9wy4lyjsrmhufkrcfhdxy27k329z66xs6xqjartzrym79ewvl0rem9r0wk8dtry43hj4nt0ghw09n60v40k3srv5uq3+3000000 \
    --out-file collateral.raw
```

2. Calculate min fee
```
cardano-cli  transaction  calculate-min-fee \
    --tx-body-file  collateral.raw \
    --tx-in-count  1 \
    --tx-out-count  2 \
    --witness-count  1 \
    --testnet-magic  1097911063 \
    --protocol-params-file  protocol-params.json
```

3. Build transaction using fees
```
cardano-cli transaction build-raw \
    --fee 180549 \
    --tx-in 09a3d2115eaf7a6bb44ca5adad62170ef737a032728c44b9e219f260085b4db8#0 \
    --tx-out addr_test1qzr8dxy9wy4lyjsrmhufkrcfhdxy27k329z66xs6xqjartzrym79ewvl0rem9r0wk8dtry43hj4nt0ghw09n60v40k3srv5uq3+94602501+"1 a90f59e9ca4020571b2f5dd6188a5a6e5a29a0e787bbbbd19b75deac.nano0002"+"1 c2b777534032e46e86b3c6b198edd8bef5a19026df95d104f44b5415.jorge" \
    --tx-out addr_test1qzr8dxy9wy4lyjsrmhufkrcfhdxy27k329z66xs6xqjartzrym79ewvl0rem9r0wk8dtry43hj4nt0ghw09n60v40k3srv5uq3+50000000 \
    --out-file collateral.raw
```

4. Sign transaction
```
cardano-cli transaction sign \
    --signing-key-file payment/payment.skey \
    --testnet-magic 1097911063 \
    --tx-body-file collateral.raw \
    --out-file collateral.signed
```

5. Submit transaction request
```
cardano-cli transaction submit \
    --tx-file  collateral.signed \
    --testnet-magic 1097911063
```

## Mint token using plutus
1. Generate minting policy using cabal:
```cabal run plutus-minting```

2. Generate policyId using cardano-cli:
```cardano-cli transaction policyid --script-file minting-policy.plutus > minting-policy.id```

3. Build the transaction
```
cardano-cli transaction build \
    --alonzo-era \
    --testnet-magic  1097911063 \
    --tx-in f10d96c6eec4e99124449fd7f18c8cc1731ecf936b9fe469663bd2b8a0d3a68b#0 \
    --tx-out addr_test1qquw7k22ae7xf772l4ljuvp7ncsq4uzsl2zlvtqlr5kf9lclzytcyjujtzmedxzjgq92kwg48my4dsdnzcmdj6eh5sxqy4j3up+2000000+"1 e17ffac1b8d529f5dd38a711bbdfd34e148cbd2044edce0038907ac1.MyNanoToken" \
    --change-address addr_test1qzr8dxy9wy4lyjsrmhufkrcfhdxy27k329z66xs6xqjartzrym79ewvl0rem9r0wk8dtry43hj4nt0ghw09n60v40k3srv5uq3 \
    --mint "1 e17ffac1b8d529f5dd38a711bbdfd34e148cbd2044edce0038907ac1.MyNanoToken" \
    --mint-script-file minting-policy.plutus \
    --mint-redeemer-value 1 \
    --metadata-json-file metadata.json \
    --tx-in-collateral 09a3d2115eaf7a6bb44ca5adad62170ef737a032728c44b9e219f260085b4db8#1 \
    --protocol-params-file  protocol-params.json \
    --out-file minting.raw
```

4. Sign the transaction
```
cardano-cli transaction sign \
    --signing-key-file payment/payment.skey \
    --testnet-magic  1097911063 \
    --tx-body-file minting.raw \
    --out-file minting.signed
```

5. Submit the transaction
```
cardano-cli transaction submit \
    --tx-file minting.signed \
    --testnet-magic 1097911063
```

6. Check your token!
> https://testnet.cardanoscan.io/tokenPolicy/e17ffac1b8d529f5dd38a711bbdfd34e148cbd2044edce0038907ac1