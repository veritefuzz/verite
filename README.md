# Verite

Verite is a smart contract fuzzing tool towards maximum profits. This repo contains the artifacts and relevant supplementary information.

## Artifact

We provide a docker image to reproduce our result.

Download link: [Link](https://mega.nz/file/qqoCwTAC#BV4F0ci17xPEr5gQ_KLdRY8WmdpAj_ma3HABkbgehkw).

Run a fuzzing experiment with

```
# Import the image
cat verite.tar.zstd | zstd -d | docker load
# See help
docker run --rm -it fse24 fuzz --help
# Run an experiment
docker run --rm -it -v /tmp/out:/tmp/out fse24 fuzz -a addr1,addr2... -b 100000 --rpc http://127.0.0.1:8545 -o /tmp/out -e https://api.bscscan.com/api -k ...
```

## Dataset

The dataset is available at [dataset.csv](./dataset.csv).

The full breakdowns of profit increasing contributions is available at [figures subfolder](./figures/).

## Supplementary Materials

### ItyFuzz FP Analysis

ItyFuzz has a high false positive rate, both in our testing and other related works[1,2]. We summarize the common reasons for false positives.

1. Off-chain oracle. ItyFuzz emulates the Uniswap V2 swapping off-chain to quote assets. This usually works for properly implemented tokens but leads to false positives if not properly handling tokens that don't conform to ERC20's semantics. For instance, some vulnerable tokens could transfer less than specified in `transfer` or revert on `transfer` even if the caller holds enough tokens. ItyFuzz's oracle fails to properly handle these corner cases when quoting assets and thus has false positives. `gss` falls into this category because ItyFuzz wrongly calculates the liquidation.
2. Impractical Attacking Conditions. ItyFuzz employs lightweight symbolic execution which could solve constraints of block numbers and timestamps. However, ItyFuzz doesn't properly check the feasibliity of the environment. For example, ItyFuzz assumes conducting the attack after __4e42__ years on `rfb`, while the age of the solar system is just around 4e9 years. This includes `rfb`, `tinu`, `lw`, `bunn`.
3. Inconsistent Account Balance. ItyFuzz will assume all accounts having `U256_MAX` balance and check if attackers can steal the funds. However, this could have side effects in two ways. First, some contracts will check if some contracts have positive balance for custom business logic. In addition, many contracts have benign functions that allow users to refund wrongly transferred ETH. Both lead to false positives in ItyFuzz. This includes `sut`, `newfi`, `lusd`, `shidoglobal`, `cfc`, `sellc03`, `cellframe`, `selltoken`, `roi`, `valuedefi` and `pancakehunny`.

It is also worth mentioning that ItyFuzz is a fast-evolving project, and all analysis subjects to the specific version. For instance, we once tested an earlier ItyFuzz (`56d6a9e075bdcefd204b511fc2e034072c73c972`). The false positives were distributed very differently and the oracle was also designed in another way.

### LLM-Assisted Actions Generation

Generating DeFi actions needs a few manual efforts. However, mainstream DeFi applications usually provide detailed instructions and samples to facilitate the usage. Here, we present how we generate an action for swapping tokens on [DODODEX](https://dodoex.io/en) using ChatGPT. Full conversation is also [attached](./conversation.txt).

Firstly, we need to grab the API document and we will use the API document of [Exchange](https://docs.dodoex.io/en/developer/contracts/dodo-v1-v2/guides/exchange) here as an example because it is one the most common usages. In addition, to help LLM better understanding the constraints of API, we also grab the unit tests (or implementation/samples) from [here](https://github.com/DODOEX/contractV2/blob/c58c067c4038437610a9cc8aef8f8025e2af4f63/test/DSP/trader.test.ts#L50C1-L104C12). Combining the two things together, we can ask ChatGPT to generate actions. This finally gives:

```solidity
// Function to sell a percentage of base tokens for quote tokens
function sellBaseTokens(address to, uint256 percentage) external returns (uint256) {
    uint256 totalBase = baseToken.balanceOf(msg.sender);
    uint256 amountToSell = (totalBase * percentage) / 100;

    require(amountToSell > 0, "Amount must be greater than zero");
    require(baseToken.transferFrom(msg.sender, address(dodo), amountToSell), "Transfer failed");
    
    uint256 receivedAmount = dodo.sellBase(to);
    return receivedAmount;
}

// Function to sell a percentage of quote tokens for base tokens
function sellQuoteTokens(address to, uint256 percentage) external returns (uint256) {
    uint256 totalQuote = quoteToken.balanceOf(msg.sender);
    uint256 amountToSell = (totalQuote * percentage) / 100;

    require(amountToSell > 0, "Amount must be greater than zero");
    require(quoteToken.transferFrom(msg.sender, address(dodo), amountToSell), "Transfer failed");
    
    uint256 receivedAmount = dodo.sellQuote(to);
    return receivedAmount;
}
```

This is almost acceptable to VERITE with a few modifications (renaming functions, adjusting `msg.sender`).

### Errata

- The `bamboo` of ItyFuzz in Table 2 should be true positive with a profit of 42 USD. This is because the exploit generated by ItyFuzz in this case can not be replayed by itself and we have to manually transform the exploit to a foundry test (for others, we use a semi-automatically approach by replaying the exploit). However, this shall not affect the conclusion of our paper.

We apologize for any inconvenience and will fix once revised.

## Reference

[1] Ye, Mingxi, et al. "Midas: Mining Profitable Exploits in On-Chain Smart Contracts via Feedback-Driven Fuzzing and Differential Analysis." Proceedings of the 33rd ACM SIGSOFT International Symposium on Software Testing and Analysis. 2024.

[2] Liang, Ruichao, et al. "Vulseye: Detect Smart Contract Vulnerabilities via Stateful Directed Graybox Fuzzing." arXiv preprint arXiv:2408.10116 (2024).
