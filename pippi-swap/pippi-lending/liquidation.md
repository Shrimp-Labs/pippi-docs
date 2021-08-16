# Liquidation

## The Liquidation process

Pippi Lending’s smart contracts provide information on the participating accounts, the amount of funds supplied by and account as well as outstanding borrows. In addition they provide the collateral factor and the relative price of each asset. A collateral factor is set for each asset and determines the percentage you can borrow against the collateral. A collateral factor is always between 0 and 1, such that the borrowing allowance is determined as _supplied\_assets x collateral\_factor_. Each borrow is thus always initially over-collateralized.

Supplied assets, collateral factor, borrowed assets and determine the Account Health. Account Health is the ratio between total USD supplied and total USD borrowed. More precisely, it is the sum of all supplied tokens, converted to USD, multiplied by the collateral factor, divided by the total sum of borrowed tokens converted to USD. When Account Health is lower than 1 it means the account has exceeded its allowance for borrowing or borrowing capacity.

Several scenarios can lead to this state: 

● The interest on the borrowed funds in higher than the interest on the collateral, and it accumulates over time

● The price of the collateral suddenly drops 

● The price of the borrowed asset suddenly shoots up

Once a liquidable account is detected, the actual liquidation is performed by calling the liquidateBorrow\(\) function in the smart contract with the following parameters 

● The address of the liquidated account 

● The repay amount paid by liquidator 

● The token of the collateral, represented by address of Pippi Lending smart contract for this token, to be repaid to the liquidator, with a 5% discount.



## Collateral Factor



| Assets | Collateral Factor |
| :---: | :---: |
| HT | 75.00%  |
| ETH | 75.00%  |
| USDT | 75.00%  |
| HUSD | 75.00%  |
| HBTC | 65.00%  |
| HBCH | 55.00%  |
| HFIL | 60.00%  |
| HDOT | 60.00%  |
| HLTC | 40.00%  |
| HPT | 15.00%  |
| MDX | 20.00%  |

