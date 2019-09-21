 
<!-- 撮合交易接口
--- -->

# 版本
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

### 输出参数

 名称|	类型|	描述
:---|:-|:-
account|string|资金账号
ycode|string|你的本地编号
errStr|string|错误信息 成功时为委托编号 

#### TransactionOrder
```  js
TransactionOrder
{
    Id (integer):  资金账号,
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
}
```

#### EntrOrder
``` js
EntrOrder
{
    orderid (integer):  委托编号,
    fundaccount (string):  资金账号,
    SecurityCode (string):  产品代码,
    price (float):  委托价格,
    volume (integer):  委托量,
    flag (integer): 委托方向 1 买 2 卖 5 撤,
    type (integer): 委托类别 1 市价 2 限价,
    category (integer): 报单方式 0 当日限价 1 即成剩撤 2 全成全撤,
    timestamp (integer): 时间戳
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

# 行情接口 `待更新`

 
