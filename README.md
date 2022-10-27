# UniswapExample  
Uniswap提供自動化做市商（AMM）模型的服務, 屏除傳統掛單簿得交易方式, 來進行貨幣兌換  
資金流動率的公式為x * y = k 依據當前市場上的數量自動進行交易  

其中核心觀念的合約內容, 就來練習一下吧  

我們利用uniswap團隊提供的範例來實作  
這邊用到的是hardhat forking mainnet的功能來進行模擬鏈上開發   

接著進入contract的部分。  
ISwapRouter
