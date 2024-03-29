 
<!-- 撮合交易接口
--- -->

# 版本
> V1.5 `2019-10-07`
+ 新增Web Socket 请求命令及返回数据模型

> V1.4 `2019-09-27`
+ 修改实时成交单为批量发送

> V1.3 `2019-09-26`
+ 修改交易委托单信息
+ 新增委托单通知命令

> V1.2 `2019-09-25`
+ 新增行情接口 socket

> V1.1 `2019-09-21`
+ 修改请求响应命令
+ 增加实时委托、成交单 数据模型

> V1.0 `2019-09-18`
+ 修改交易接口 请求命令及返回数据
 

# 数据包协议
---
Cmd|Data Length|Data
-|-|-

> 1. **Cmd** 指的是数据包的命令,可以指定数据包用来的目的, 对应命令请看下文<br>
    Cmd 占用 2 字节
> 2. **Data Length** 指的是数据的长度<br> Data Length 占用 4 字节 
> 3. **Data** 数据体 压缩与加密只对数据体进行操作<br> Data 占用 N 字节


# 交易接口
 
## 请求/响应命令
---

Cmd|Data|描述|返回数据
:---|:--- |:--- |:--
0  |[]|心跳包|[]
1  |[account,code,type,price,volume,ycode]|委买|[account,ycode, errStr]
2  |[account,code,type,price,volume,ycode]|委卖|[ account, ycode, errStr]
3  |[account,entrustNo,ycode]|撤委托|[ account, ycode, errStr]
4  |[code,transactionno]`被动响应`|实时成交单|[TransactionOrder](/?id=TransactionOrder)
5  |[code,entrustno]`被动响应`|实时委托单|[EntrOrder](/?id=EntrOrder)
6  |`被动响应`|委托单状态通知|[entrustno,entrusstatus]


### 请求参数

名称|	类型|	描述
:---|:-|:-
account|string|资金账号
code|string|产品代码
type|int| 委托类别: `市价 1` ` 限价 2` 
price|float|价格 `type为市价时可为 0`
volume|int|数量
ycode|string|你的本地编号
entrustno|string|委托编号
transactionno|string|成交编号
entrusstatus|int|委托完结状态 `委托中 0` `已成交 1` `废单 2` `已撤 3` `撤单失败 4` 

### 输出参数

 名称|	类型|	描述
:---|:-|:-
account|string|资金账号
ycode|string|你的本地编号
errStr|string|错误信息 成功时为委托编号 

#### TransactionOrder
```  js
[
    Id (integer):  id,
    FundAccount (string):  资金账号,
    SecurityCode (string):  产品代码,
    SecurityName (string):  产品名称,
    EntrustNo (integer):  委托编号,
    BuySellMark (integer):  买卖标志 1 买 2 卖 5 撤,
    BuySellMarkStr (string):  买卖标志说明 ,
    EntrustType (integer):  委托类别  1 市价 2 限价,
    TransactionNo (integer):  成交编号,
    TransactionPrice (float):  成交价格,
    TransactionCount (integer):  成交数量,
    Amount (float):  发生金额,
    CreateDate (string):  创建日期,
    Date (string):  日期 [yyyyMMdd],
    Time (string):  时间 [HHmmss]
]
```

#### EntrOrder
``` js
EntrOrder
{
    Id (integer):  委托编号,
    FundAccount (string):  资金账号,
    SecurityCode (string):  产品代码,
    SecurityName (string):  产品名称,
    SecurityType (int): 产品类型
    EntrustPrice (float):  委托价格,
    EntrustCount (integer):  委托量,
    TransactionPrice (float):  成交价格,
    TransactionCount (integer):  成交数量,
    Amount (float):  成交金额,
    Status (int):  状态,
    BuySellMark (integer): 委托方向 1 买 2 卖 5 撤,
    PriceType (integer): 委托类别 1 市价 2 限价,
    Category (integer): 报单方式 0 当日限价 1 即成剩撤 2 全成全撤,
    CreateDate (integer): 日期
}

```

### 返回数据
``` js
 ResultInfo {
    code (integer): 错误码 0 失败 1 成功 ,
    cmd (Cmd): 命令 , 
    data (object): 返回数据
}
```

# 行情接口
---
## Socket

### 请求/响应命令
Cmd|Data|描述|返回数据
:---|:--- |:--- |:--
0  |[]|心跳包 频率:40秒至60秒一次|{"code":1,"data":"ok","islast":true}
1  |[ActionType,[Code]]|订阅行情 |
2  |[ActionType,["Code_KlineType"]]|订阅K线|{"code":1,"data":"订阅k线成功","islast":true}
3  |Code|订阅实时成交|{"code":1,"data":"订阅实时成交成功","islast":true}
6  |`被动响应`|行情推送|[SecurityQuotes](/?id=SecurityQuotes)
7  |`被动响应`|k线推送|[Kline](/?id=Kline)
8  |`被动响应`|成交推送|[TransactionData](/?id=TransactionData)
9  |`被动响应`|分时推送|[KlineMk](/?id=Kline)
10 |?user=test&pwd=test123789<br>&mode=Binary&zilb=no&encryption=no|登录 `登录消息不需要压缩加密`|

#### 请求参数

名称|	类型|	描述
:---|:-|:-
ActionType|int|订阅操作类型 `覆盖 0` ` 增加 1`  ` 减少 2` ` 取消 3`
code|string|产品代码
KlineType|int| k线类型: `1分钟 0` `5分钟 1` `15分钟 2` `30分钟 3` `60分钟 4 ` `日 5` `周 6` `月 7` `分时 8` `120分钟 9` `季 10` `年 11`
user|string| 登录名
pwd|string| 登录密码
zlib|string| 数据压缩: `压缩 yes` `不压缩 no`
encryption|string| 数据加密,目前只支持AES加密算法,: `加密 yes` `不加密 no`
mode|string|数据收发模式: `Binary 二进制` `Text 文本 `

#### 输出参数
##### SecurityQuotes
```  js
[
    SecurityCode (string): 产品代码,
    SecurityType (int): 产品类型
    DateStr (string): 时间戳,
    MaxPx (float): 涨停价,
    MinPx (float): 跌停价,
    Price (float): 最新价,
    PreClose (float): 昨收价,
    Volume(int): 现手,
    TotalVolumeTrade(int): 成交总量（笔数）,
    TotalValueTrade(int): 总成交金额,
    Open(float): 开盘,
    High(float): 最高,
    Low(float): 最低,
    Close(float): 收盘, 
    LastUpdateMillisec(int): 行情发送时间 10位,
    TotalBuyQty(int): 买入总量,
    TotalSellQty(int): 卖出总量, 
    BuyPriceQueueList(arrar): 买入价格队列,
    BuyOrderQtyQueueList(array): 买入数量队列, 
    SellPriceQueueList(arrar): 卖出价格队列,
    SellOrderQtyQueueList(array): 卖出数量队列, 
    UpDowns(float): 涨跌,
    UpDownsRate(float): 涨幅,
]
```
##### TransactionData
```  js
[
    SecurityCode (string): 产品代码,
    SecurityType (int): 产品类型
    Date (string): 时间戳,
    Time (string): 成交时间,
    Price (float): 价格,
    Volume(int): 成交量,
    Lot (int): 笔数,
    bs (string): 买卖,
    Trade (float): 成交额
]
```
##### Kline
```  js
[
    SecurityCode (string): 产品代码,
    SecurityType (int): 产品类型
    KlineType (int): k线类型
    Date (string): 日期,
    Open (float): 开盘,
    High (float): 最高,
    Low(float): 最低,
    Close (float): 收盘,
    Volume (int): 成交量,
    Turnover (float): 成交额,
    PrecClose (float): 昨收 只有分时会有
    
]
```

## WebSocket

### 请求/响应命令
Cmd|Data|描述|返回数据
:---|:--- |:--- |:--
-1  |`被动响应`|异常错误|{"code":0,"data":"错误信息","islast":true}
0  |[]|心跳包 频率:40秒至60秒一次|{"code":1,"data":"ok","islast":true}
1  |[ActionType,\[Code\] ]|订阅行情 |
2  |[ActionType,["Code_KlineType"]]|订阅K线|{"code":1,"data":"订阅k线成功","islast":true}
3  |Code|订阅实时成交|{"code":1,"data":"订阅实时成交成功","islast":true}
6  |`被动响应`|行情推送|[实时行情](#实时行情)
7  |`被动响应`|k线推送|[实时k线](#实时K线分时)
8  |`被动响应`|成交推送|[实时成交](#实时成交)
9  |`被动响应`|分时推送|[实时分时](#实时K线分时 )


> 用例 ws://127.0.0.1:21219/mode=Binary&zilb=yes&encryption=yes&user=user&pwd=pwd

> 请求json格式为:

``` js
    {
        "Cmd": Cmd,
        "Data":Data
    }

```
#### 请求参数

名称|	类型|	描述
:---|:-|:-
ActionType|int|订阅操作类型 `覆盖 0` ` 增加 1`  ` 减少 2` ` 取消 3`
code|string|产品代码
KlineType|int| k线类型: `1分钟 0` `5分钟 1` `15分钟 2` `30分钟 3` `60分钟 4 ` `日 5` `周 6` `月 7` `分时 8` `120分钟 9` `季 10` `年 11`
user|string| 登录名
pwd|string| 登录密码
zlib|string| 数据压缩: `压缩 yes` `不压缩 no`
encryption|string| 数据加密,目前只支持AES加密算法,: `加密 yes` `不加密 no`
mode|string|数据收发模式: `Binary 二进制` `Text 文本 `

#### 输出参数
 
##### 实时行情
```  js
[
    SecurityCode (string): 产品代码,
    SecurityType (int): 产品类型
    DateStr (string): 时间戳,
    MaxPx (float): 涨停价,
    MinPx (float): 跌停价,
    Price (float): 最新价,
    PreClose (float): 昨收价,
    Volume(int): 现手,
    TotalVolumeTrade(int): 成交总量（笔数）,
    TotalValueTrade(int): 总成交金额,
    Open(float): 开盘,
    High(float): 最高,
    Low(float): 最低,
    Close(float): 收盘, 
    LastUpdateMillisec(int): 行情发送时间 10位,
    BuyPriceQueueList(arrar): 买入价格队列,
    BuyOrderQtyQueueList(array): 买入数量队列, 
    SellPriceQueueList(arrar): 卖出价格队列,
    SellOrderQtyQueueList(array): 卖出数量队列
]
```

##### 实时成交
```  js
[
    SecurityCode (string): 产品代码,
    SecurityType (int): 产品类型
    Date (string): 时间戳,
    Time (string): 成交时间,
    Price (float): 价格,
    Volume(int): 成交量,
    Lot (int): 笔数,
    bs (string): 买卖,
    Trade (float): 成交额
]
```
##### 实时K线分时 
```  js
[
    SecurityCode (string): 产品代码,
    SecurityType (int): 产品类型
    KlineType (int): k线类型
    Date (string): 日期,
    Open (float): 开盘,
    High (float): 最高,
    Low(float): 最低,
    Close (float): 收盘,
    Volume (int): 成交量,
    Turnover (float): 成交额,
    PrecClose (float): 昨收 只有分时会有
]
```

 
