 
<!-- 撮合交易接口
--- -->

# 数据包协议
---
Cmd|Data Length|Data
-|-|-

>* Cmd指的是数据包的命令,可以指定数据包用来的目的, 对应命令请看下文
    Cmd 占用 2 字节
* Data Length指的是数据的长度
    Data Length 占用 4 字节 
* Data 数据体
Data 占用 N 字节,压缩与加密只对数据体进行操作

 
# 请求/响应命令
---
命令(Key)	|值	|描述	|请求用例	|返回数据
-|-|-|-|-
Heartbeat|0|心跳包|	[]|	{}
BuyOrder|1|买|[account,code,type,price,volume,ycode]|[ account, youcode, errStr]
SellOrder|2|卖|[资金账号，产品编码，[委托类别](#委托类别),价格，数量，编号]|
CancelOrder|3|撤|[资金账号，委托编码，编号]|
TransactionData|4|实时成交|被动响应|
EntrustData|5|市价实时委托|被动响应|

# 参数说明
---
>[资金账号，产品编码，[委托类别](#委托类别),价格，数量，编号]
 
> 市价 = 1,限价 = 2,本方最优 = 3

>委托类别
```委托类别
市价 = 1,限价 = 2,本方最优 = 3
```
 

```




















```











 # 委托类别
 
