# Borrowing Assets

Borrowing can be done with a user interface, or with **code**. Before we walk through the steps of a borrowing sequence, let’s cover some **key concepts**.

1. **Collateral** — In order to borrow crypto from the Pippi Lending, users need to first supply another type of crypto as collateral. This is provided using the same **mint** function used for supplying assets. Supplied collateral assets earn interest while in the protocol, but users cannot redeem or transfer assets while they are being used as collateral.
2. **Collateral Factor** — The maximum amount users can borrow is limited by the collateral factors of the assets they have supplied. For example, if a user supplies 100 DAI as collateral, and the posted collateral factor for DAI is 75%, then the user can borrow at most 75 DAI worth of other assets at any given time. Each asset on Pippi Lending can have a different collateral factor. Collateral factors for each asset can be fetched using the Comptroller contract. 
3. **Enter Markets** — An asset that is supplied to the protocol is not usable as collateral initially. In order to inform the protocol that you wish to use an asset as collateral, you must “enter the market” for that asset. An account can enter multiple markets at one time.
4. **Calculating Account Liquidity** — How can you tell what your maximum allowed borrow amount is when you have multiple collateral assets with differing collateral factors? The Comptroller contract provides an easy to use function that calculates your account’s liquidity, which is a USD-denominated value of the maximum allowed borrow amount. You should never borrow this much at once because your account would instantly be liquidated as soon as the protocol’s “accrue interest” operation is executed.
5. **The Open Price Feed** — How can the protocol calculate the maximum borrow, or if a user’s account is insolvent? Pippi Lending has its own Price Feed contract that has the current exchange rates of all supported assets. The Open Price Feed uses price feeds from several highly liquid exchanges to find the median price, and has rules coded into the contract to ensure the integrity of its reported prices. 
6. **The Borrow Balance** — This is the sum of a user’s current borrowed amount plus the interest that needs to be repaid. This is calculated with an easy to use function in each pToken contract.
7. **The Borrow Rate** — Borrowers owe the prevailing interest rate of the asset they are borrowing. This is done by adding the Borrow Rate to the account’s **Borrow Balance** every Ethereum block. While a borrow is open, the Borrow Balance is **ever-increasing**. The borrow and interest is never required to be repaid unless the borrower becomes insolvent; otherwise, the borrower can choose to repay some or all of their borrow whenever they choose.
8. **Liquidation** — A borrowing account becomes insolvent when the Borrow Balance exceeds the amount allowed by the collateral factor. When an account becomes insolvent, other users can repay a portion of its outstanding borrow in exchange for a portion of its collateral, with a liquidation incentive — currently set at 8%. The liquidation incentive means that liquidators receive the borrower’s collateral at a 8% discount to market price. Having your account liquidated is **bad** because you lose some of your collateral.
9. **Repaying a Borrow** — Borrows can be repaid using a function on the respective pToken contract. Once a borrow has been repaid, the account’s collateral can be entirely redeemed or transferred. There are also functions in the pToken contracts to repay a borrow on behalf of another account.

Let’s dive into borrowing examples using **JavaScript** or **Solidity**. The order of operations for borrowing are:

### Supply Collateral <a id="2a26"></a>

1. Supply one or many supported crypto assets to Pippi Lending, as **collateral**, by calling the **mint** function in the pToken smart contract. You will receive pTokens in return for your asset. Note that collateral earns interest, even while a borrow is open against it.
2. Enter the market of the supplied asset or assets by calling the Comptroller’s **enterMarkets** function. Call this function and pass the address of the pToken contract of the asset in which you want to use as **collateral**.
3. Choose an asset you want to **borrow** from the supported assets listed on the Pippi Lending Markets.

### Calculate the amount to borrow <a id="7e90"></a>

1. Use the Open Price Feed contract’s **price** function to get the price in USD of the asset you wish to borrow. We’ll need this to find out just how much we can borrow.
2. Call **getAccountLiquidity** on the Comptroller to get the USD value of your account’s liquidity.
3. Calculate the maximum amount you can borrow of the asset. Divide the account’s liquidity by the price of the asset you wish to borrow. 

### Borrow <a id="3ae3"></a>

1. Call the corresponding pToken contract’s **borrow** function, passing an amount that is less than your maximum allowed borrow.
2. Do things with your borrowed asset while making sure your account does not go **insolvent**.
3. When you’re ready, repay your borrow using the pToken’s **repayBorrow** function.

This can be done solely in **JavaScript** using Web3.js JSON RPC or with a proxy smart contract written in **Solidity**.

