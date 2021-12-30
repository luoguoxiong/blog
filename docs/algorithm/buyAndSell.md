# 买卖股票问题

### 一、 单次买卖最大利润

```shell
输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
  let result = 0;
  let buyPrices = prices[0]
  for(let i = 1;i< prices.length;i++){
    const sell = prices[i]
    if(sell<buyPrices){
      buyPrices = sell
    }
    result = Math.max(result,sell-buyPrices)
  }
  return result
}
```

### 二、 多次买卖股票最大利润

```shell
输入: prices = [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
```

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
  let result = 0;
  for(let i = 0;i < prices.length - 1;i++){
    result += Math.max(prices[i + 1] - prices[i], 0);
  }
  return result;
};
```

### 三、多次买卖股票最大利润含手续费

```shell
输入：prices = [1, 3, 2, 8, 4, 9], fee = 2
输出：8
解释：能够达到的最大利润:  
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8
```

```js
/**
 * @param {number[]} prices
 * @param {number} fee
 * @return {number}
 */
var maxProfit = function(prices, fee) {
  let buyPrice = prices[0];
  let result = 0;
  for(let i = 1;i < prices.length;i++){
    const sellPrice = prices[i];
    if(sellPrice < buyPrice){
      buyPrice = sellPrice;
      continue
    }
    if(sellPrice >= buyPrice && sellPrice === buyPrice + fee){
      continue;
    }
    if(sellPrice > buyPrice + fee){
      result += (sellPrice - buyPrice - fee);
      buyPrice = sellPrice - fee;
    }
  }
  return result;
};
```

