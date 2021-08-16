# Protocol Math

The Pippi Lending contracts use a system of exponential math, Exponential.sol, in order to represent fractional quantities with sufficient precision.

Most numbers are represented as a mantissa, an unsigned integer scaled by 1 \* 10 ^ 18, in order to perform basic math at a high level of precision.

#### pToken and Underlying Decimals

Prices and exchange rates are scaled by the decimals unique to each asset; pTokens are HRC-20 tokens with 8 decimals, while their underlying tokens vary, and have a public member named decimals.

| pToken | pToken Decimals | Underlying | Underlying Decimals |
| :--- | :--- | :--- | :--- |
| pHT | 8 | HT | 18 |
| pETH | 8 | ETH | 18 |
| pUSDT | 8 | USDT | 18 |
| pHUSD | 8 | HUSD | 8 |
| pHBTC | 8 | HBTC | 18 |
| pHBCH | 8 | HBCH | 18 |
| pHFIL | 8 | HFIL | 18 |
| pHDOT | 8 | HDOT | 18 |
| pHLTC | 8 | HLTC | 18 |
| pHPT | 8 | HPT | 18 |
| pMDX | 8 | MDX | 18 |

#### Interpreting Exchange Rates

The pToken Exchange Rate is scaled by the difference in decimals between the pToken and the underlying asset.

```text
onePTokenInUnderlying = exchangeRateCurrent / (1 * 10 ^ (18 + underlyingDecimals - pTokenDecimals))
```

Here is an example of finding the value of 1 pETH in ETH with Web3.js JavaScript.

```text
const pTokenDecimals = 8; // all pTokens have 8 decimal places
const underlying = new web3.eth.Contract(erc20Abi, batAddress);
const pToken = new web3.eth.Contract(pTokenAbi, cBatAddress);
const underlyingDecimals = await underlying.methods.decimals().call();
const exchangeRateCurrent = await pToken.methods.exchangeRateCurrent().call();
const mantissa = 18 + parseInt(underlyingDecimals) - pTokenDecimals;
const onePTokenInUnderlying = exchangeRateCurrent / Math.pow(10, mantissa);
console.log('1 pHT can be redeemed for', onePTokenInUnderlying, 'BAT');
```

There is no underlying contract for HT, so to do this with pHT, set underlyingDecimals to 18.

To find the number of underlying tokens that can be redeemed for pTokens, multiply the number of pTokens by the above value onePTokenInUnderlying.

```text
underlyingTokens = pTokenAmount * onePTokenInUnderlying
```

#### Calculating Accrued Interest

Interest rates for each market update on any block in which the ratio of borrowed assets to supplied assets in the market has changed. The amount interest rates are changed depends on the interest rate model smart contract implemented for the market, and the amount of change in the ratio of borrowed assets to supplied assets in the market.

See the interest rate data visualization notebook on Observable to visualize which interest rate model is currently applied to each market.

Interest accrues to all suppliers and borrowers in a market when any heco address interacts with the market’s pToken contract, calling one of these functions: mint, redeem, borrow, or repay. Successful execution of one of these functions triggers the accrueInterest method, which causes interest to be added to the underlying balance of every supplier and borrower in the market. Interest accrues for the current block, as well as each prior block in which the accrueInterest method was not triggered \(no user interacted with the pToken contract\). Interest compounds only during blocks in which the pToken contract has one of the aforementioned methods invoked.

Here is an example of supply interest accrual:

Alice supplies 1 ETH to the Pippi Lending protocol. At the time of supply, the supplyRatePerBlock is 37893605 Wei, or 0.000000000037893605 ETH per block. No one interacts with the pETH contract for 3 heco blocks. On the subsequent 4th block, Bob borrows some ETH. Alice’s underlying balance is now 1.000000000151574420 ETH \(which is 37893605 Wei times 4 blocks, plus the original 1 ETH\). Alice’s underlying ETH balance in subsequent blocks will have interest accrued based on the new value of 1.000000000151574420 ETH instead of the initial 1 ETH. Note that the supplyRatePerBlock value may change at any time.

#### Calculating the APY Using Rate Per Block

The Annual Percentage Yield \(APY\) for supplying or borrowing in each market can be calculated using the value of supplyRatePerBlock \(for supply APY\) or borrowRatePerBlock \(for borrow APY\) in this formula:

```text
Rate = pToken.supplyRatePerBlock(); // Integer
Rate = 37893566
ETH Mantissa = 1 * 10 ^ 18 (ETH has 18 decimal places)
Blocks Per Day = 6570 (13.15 seconds per block)
Days Per Year = 365

APY = ((((Rate / ETH Mantissa * Blocks Per Day + 1) ^ Days Per Year)) - 1) * 100
```

Here is an example of calculating the supply and borrow APY with Web3.js JavaScript:

```text
const ethMantissa = 1e18;
const blocksPerDay = 6570; // 13.15 seconds per block
const daysPerYear = 365;

const pToken = new web3.eth.Contract(cEthAbi, cEthAddress);
const supplyRatePerBlock = await pToken.methods.supplyRatePerBlock().call();
const borrowRatePerBlock = await pToken.methods.borrowRatePerBlock().call();
const supplyApy = (((Math.pow((supplyRatePerBlock / ethMantissa * blocksPerDay) + 1, daysPerYear))) - 1) * 100;
const borrowApy = (((Math.pow((borrowRatePerBlock / ethMantissa * blocksPerDay) + 1, daysPerYear))) - 1) * 100;
console.log(`Supply APY for ETH ${supplyApy} %`);
console.log(`Borrow APY for ETH ${borrowApy} %`);
```

