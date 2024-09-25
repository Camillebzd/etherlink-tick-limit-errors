# Etherlink Tick Limit Errors

This repository aims to provide repeatable examples detailing the Etherlink tick limit issues. We will use a script inspired by [IguanaDEX]("https://iguanadex.com/") as an example to highlight the engineering problems on the blockchain.

## Summary

The current tick limit on Etherlink is too low for any advanced on-chain computation to be executed. In our example, we call a multicall contract which is required by IguanaDEX to simulate swaps and provide price quotes to users on the frontend. This contract loops over an array containing all calls to be executed by the transaction, which should finally return an array with all results.

However, this call fails on Etherlink Mainnet and Etherlink Testnet. We receive the following error when more than **16** calls are included:

```bash
ProviderError: The transaction would exhaust all the ticks it is allocated. Try reducing its gas consumption or splitting the call in multiple steps, if possible.
```

This number is too low, and in the short term should be able to support at least **40**, as is possible on other EVM chains, such as BNB Chain here.

## Setup

Create a `.env` and add your info:
```
PRIVATE_KEY=

ETHERLINK_RPC_URL=https://node.ghostnet.etherlink.com
ETHERLINK_API_KEY=YOU_CANCOPY_ME

BSC_TESTNET_URL=https://bsc-testnet.publicnode.com
BSCSCAN_API_KEY=YOU_CANCOPY_ME
```

Finally, run:
```
npm install
```

## Running Tests

**Etherlink Testnet**
```
npx hardhat run scripts/callQuoterIguana.ts --network etherlinkTestnet
```

The result is as follows:

```bash
$ npx hardhat run scripts/callQuoterIguana.ts --network etherlinkTestnet

--- before call ---
Number of iteration: 20
ProviderError: The transaction would exhaust all the ticks it
    is allocated. Try reducing its gas consumption or splitting the call in
    multiple steps, if possible.
    at HttpProvider.request (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/hardhat/src/internal/core/providers/http.ts:107:21)
    at processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async HardhatEthersProvider.estimateGas (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/@nomicfoundation/hardhat-ethers/src/internal/hardhat-ethers-provider.ts:246:27)
    at async /Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/@nomicfoundation/hardhat-ethers/src/signers.ts:235:35
    at async Promise.all (index 0)
    at async HardhatEthersSigner._sendUncheckedTransaction (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/@nomicfoundation/hardhat-ethers/src/signers.ts:256:7)
    at async HardhatEthersSigner.sendTransaction (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/@nomicfoundation/hardhat-ethers/src/signers.ts:125:18)
    at async send (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/ethers/src.ts/contract/contract.ts:313:20)
    at async Proxy.multicallWithGasLimitation (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/node_modules/ethers/src.ts/contract/contract.ts:352:16)
    at async sendTransaction (/Users/camillebouzerand/Documents/Code/Iguana/iguana-smart-router-errors/scripts/callQuoterIguana.ts:198:14)
```

**BNB Testnet**
```
npx hardhat run scripts/callQuoterIguanaBSC.ts --network bscTestnet
```

The result is as follows:

```bash
$ npx hardhat run scripts/callQuoterIguanaBSC.ts --network bscTestnet

--- before call ---
Number of iteration: 40
--- call done ---
```

### Etherlink testnet Info

The contracts used are:
- [PancakeInterfaceMulticallV2](https://testnet.explorer.etherlink.com/address/0xa318d3b9Af779Fae6429cA689bc2241d01F4C8D0), used to batch the transactions in one atomic transaction
- [QuoterV2](https://testnet.explorer.etherlink.com/address/0x6e8432F0Ed242fABfA481dd449407b0f724d8D03), used to estimate the number of token will be received

### BSC testnet Info

The contracts used are:
- [PancakeInterfaceMulticallV2](https://testnet.bscscan.com/address/0xeeF6ff30cF5D5b8aBA0DE16A01d17A0697a275b5), used to batch the transactions in one atomic transaction
- [QuoterV2](https://testnet.bscscan.com/address/0xbC203d7f83677c7ed3F7acEc959963E7F4ECC5C2), used to estimate the number of token will be received
