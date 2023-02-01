**CidNFT**
- L225/279/280 - Instead of doing i - 1 or i + 1 you can do ++i or --i generating a lower cost of gas.

- L186/187/247/248 - A variable is created in memory but it is only used once, therefore creating it generates an unnecessary extra gas cost, it should be eliminated and used directly where the operation is needed.
