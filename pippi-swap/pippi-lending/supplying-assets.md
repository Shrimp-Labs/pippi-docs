# Supplying Assets

The currently supported assets are listed here [https://docs.pippi.finance/pippi-swap/pippi-lending/protocol-math](https://docs.pippi.finance/pippi-swap/pippi-lending/protocol-math). Based on the different implementation of HT and HRC-20 tokens, we have to utilize two similar processes:

* The HT supply method
* The HRC20 token supply method

Like mentioned earlier, when someone supplies an asset to the protocol, they are given p**Tokens** in exchange. The method for getting **pHT** is different from the method for getting **pETH**, **pUSDT**, ****or any other pToken for an HRC-20 asset.

When supplying HT to the Pippi Lending, an application can send HT directly to the payable **mint** function in the pHT contract. Following that mint, pHT is minted for the wallet or contract that invoked the mint function. Remember that if you are calling this function from another smart contract, that contract needs a **payable** function in order to receive HT when you redeem the pTokens later.

The operation is slightly different for pHRC20 tokens. In order to mint pHRC20 tokens, the invoking wallet or contract needs to first call the **approve** function on the **underlying token’s contract**. All HRC20 token contracts have an **approve** function.

The approval needs to indicate that the corresponding pToken contract is permitted to take _up to the specified amount_ from the sender address. Subsequently, when the **mint** function is invoked, the pToken contract retrieves the indicated amount of underlying tokens from the sender address, based on the prior approve call.

