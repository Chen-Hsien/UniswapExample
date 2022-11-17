# UniswapExample  
Uniswap提供自動化做市商（AMM）模型的服務, 屏除傳統掛單簿得交易方式, 來進行貨幣兌換  
資金流動率的公式為x * y = k 依據當前市場上的數量自動進行交易  

其中核心觀念的合約內容, 就來練習一下吧  

我們利用uniswap團隊提供的範例來實作  

申請Alchemy API key，hardhat forking mainnet的功能來進行模擬鏈上開發   
<img width="1468" alt="image" src="https://user-images.githubusercontent.com/24216536/202123443-e30dff61-a906-40fa-aa69-3ccc6d63181f.png">   
run 起來後 可會提供10組Local的帳號給我們模擬以太坊開發～。  
<img width="585" alt="image" src="https://user-images.githubusercontent.com/24216536/202131155-31526e9c-ff4e-4d41-bb0a-10a29b1c5d7e.png">   

(ISwapRouter)[https://docs.uniswap.org/protocol/reference/periphery/interfaces/ISwapRouter] 是uniswap V3 中已經定義好的Interface可以根據此耕快速地進行Swap功能的開發.  

1. hard code要互相進行轉換的DAI, WETH地址, 手續費, 這邊甜的地址因為是從主網分岔出來，所以填上Etherscan上找到的地址即可。  
```Solidity
    address public constant DAI = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    address public constant WETH9 = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    uint24 public constant feeTier = 3000;
```

2. swapWETHForDAI內實作貨幣的交換，
safeTransFrom ：為將想兌換的金額傳入合約中
safeApprove ：允許SwapRouter花費特定amount近來swap的動作
ExactInputSingleParams :memory 代表將此結果存在區塊鏈上(花費gas高)，內容則是控制傳入的token,傳出的token以及數量等等即可。   
```Solidity
function swapWETHForDAI(uint amountIn) external returns (uint256 amountOut) {

        // Transfer the specified amount of WETH9 to this contract.
        TransferHelper.safeTransferFrom(WETH9, msg.sender, address(this), amountIn);
        // Approve the router to spend WETH9.
        TransferHelper.safeApprove(WETH9, address(swapRouter), amountIn);
        // Create the params that will be used to execute the swap
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: WETH9,
                tokenOut: DAI,
                fee: feeTier,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountIn: amountIn,
                amountOutMinimum: 0,
                sqrtPriceLimitX96: 0
            });
        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
        return amountOut; 
    }
```

3. 測試Contract，這邊使用Chai套件以及Hardhat，其他測試套件還有像Mocha等.  
測試中需要做四個動作.  
1. 部署SimpleSwap.sol這個合約.  
2. 檢查test wallet的餘額.  
3. 呼叫swapWETHForDAI，進行貨幣交換.  
4. 確認DAI Balance有因此增加.  

利用hardhat ethers提供的function 進行合約的部署.  
```Solidity
/* Deploy the SimpleSwap contract */
const simpleSwapFactory = await ethers.getContractFactory('SimpleSwap')
const simpleSwap = await simpleSwapFactory.deploy(SwapRouterAddress)
await simpleSwap.deployed()
```

這邊將錢包中的ETH打包成WETH跟合約溝通   
```Solidity
/* Connect to WETH and wrap some eth  */
let signers = await hre.ethers.getSigners()
const WETH = new hre.ethers.Contract(WETH_ADDRESS, ercAbi, signers[0])
const deposit = await WETH.deposit({ value: hre.ethers.utils.parseEther('10') })
await deposit.wait()
```

同樣取得DAI地址中的餘額.  
```Solidity
/* Check Initial DAI Balance */
const DAI = new hre.ethers.Contract(DAI_ADDRESS, ercAbi, signers[0])
const expandedDAIBalanceBefore = await DAI.balanceOf(signers[0].address)
const DAIBalanceBefore = Number(hre.ethers.utils.formatUnits(expandedDAIBalanceBefore, DAI_DECIMALS))
```

Approve 用來允許傳入的地址使用用戶的1WETH,並將其中的0.1WETH利用方才的simpleSwap轉換成DAI  
```Solidity
/* Approve the swapper contract to spend WETH for me */
await WETH.approve(simpleSwap.address, hre.ethers.utils.parseEther('1'))
/* Execute the swap */
const amountIn = hre.ethers.utils.parseEther('0.1')
const swap = await simpleSwap.swapWETHForDAI(amountIn, { gasLimit: 300000 })
swap.wait()
```

最後在寫一個檢查用的判斷式看轉換後的DAI餘額有沒有大於轉換前就ok囉.  
```Solidity
/* Test that we now have more DAI than when we started */
expect(DAIBalanceAfter).is.greaterThan(DAIBalanceBefore)
```

在console中下，就可以利用本機分岔的以太坊來進行本次撰寫程式的測試囉！   
```Solidity
npx hardhat test --network localhost
```


