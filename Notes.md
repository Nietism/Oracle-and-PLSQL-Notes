# 学习笔记

![Oracle 图标](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1564591842751&di=3cb0b9bc69df78d08816abe6101c7fc1&imgtype=0&src=http%3A%2F%2Fwww.soomal.com%2Fimages%2Fdoc%2F20110713%2F00012096.jpg "Oracle logo")

## 基础知识

### 第1章 基础知识

#### 1.1 数据库基础

##### 1.1.1 什么是数据库

&#160; &#160; &#160; &#160;**数据库**（Database）指<u>以某种有组织的方式存储的数据集合</u>。可以将它看成一个档案柜，档案柜只是存储数据的物理位置，而不管数据是什么以及它们是怎样组织的。

> **数据库**
>
>  用于存储有组织的数据的容器（通常是文件或文件集）。

> **混淆**
>
> 人们经常使用**数据库**（Database）这个词来指代它们正在运行的数据库软件。这是不正确的。数据库软件实际上被称为**数据库管理系统**（Database Management System, DBMS）。数据库是通过DBMS创建和操作额容器。数据库可能是存储在硬盘驱动器上的文件，也可能不是。一般而言，这并不重要，因为用户从不会以任何方式直接访问数据库，用户总是使用DBMS，通过它来为用户访问数据库。

##### 1.1.2 表

&#160; &#160; &#160; &#160;当在档案柜中存储信息时，不会把信息简单地扔进抽屉里。相反，将在档案柜里创建文件，然后把相关的数据归档在特定的文件中。

&#160; &#160; &#160; &#160;在数据库世界里，这个文件称之为**表**（Table）。表是<u>结构化的文件，可以存储特定类型的数据</u>。表可能包括顾客列表、产品目录，或者其他人的信息列表。

> **表**
>
> 特定类型数据的结构化列表。

&#160; &#160; &#160; &#160;这里的关键是：存储在表中的数据是一种数据类型或者一份列表。永远不会有人把顾客列表和订单列表存储在同一张数据库表中。如果这样做，将使得之后的检索和访问难以进行。正常情况是，你将创建两张表，每个表用于存储一份列表。

&#160; &#160; &#160; &#160;数据库中的每一个表都具有一个用于标识它的名称，这个名称总是唯一的，意味着该数据库中没有其他的表可以具有相同的名称。

> **注意：表名称**
>
> 使表名称成为唯一的实际上是几个内容的组合，包括数据库名称和表名称。这意味着尽管不能在相同的数据库中多次使用相同的表名称，但是肯定可以在不同的数据库中重用表名称。

## 应用实例

### Example 01

```SQL
--
SELECT 
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间, 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间, 
  STATUS.DESCRIPTION 状态, 
  COUNT(OD.ORDERKEY) 单数, 
  SUM(OD.TOTALQTY) 件数, 
  OD.RTXSHOPNAME
FROM 
  ORDERS OD, 
  (SELECT
     O.CODE,
    T.DESCRIPTION 
  FROM
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE
    T.CODE = O.CODE AND
    T.TBLNAME='ORDERSTATUSSETUP' AND
    T.LOCALE = 'ZH' AND
    T.COLUMNNAME ='DESCRIPTION'
  ) STATUS
WHERE 
  (OD.TYPE = '85' ) AND 
  STATUS.CODE = OD.STATUS AND 
  NVL(OD.RTXCANCELMARK,' ') = ' ' AND 
  OD.STATUS < '95'
  --AND TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= '2019-07-03'
  --AND TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') >= '2019-06-25'
GROUP BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD'), 
  STATUS.DESCRIPTION, 
  OD.RTXSHOPNAME
ORDER BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD');
```

- `ORDERS`表中，订单的取消标记`RTXCANCELMARK`和`RTXCANCELBACK`两个项存储，其中与WMS出货单页面上的“取消标识”对应的是`RTXCANCELBACK`。`RTXCANCELMARK`的取值和`RTXCANCELBACK`取值如下：

| RTXCANCELMARK | RTXCANCELBACK |
| :-----------: | :-----------: |
|      98       |       Y       |
|               |       N       |

- 某订单若被标记取消，该订单的`RTXCANCELMARK`字段为98。`RTXCANCELBACK`字段为Y表示YES即已取消；若未被标记取消，该订单的`RTXCANCELMARK`字段为一个空字符串即'&#160;&#160;'，搜索时可以用'&#160;&#160;'作为关键字搜索。
- `nvl()`作为Oracle/PLSQL中的一个函数，其格式为：`nvl(string1, replace_with)`，它的功能如下：如果string1为`NULL`，则NVL函数返回`replace_with`的值，否则返回`string1`的值。
  在本SQL语句段的过滤条件中，这一小句这样写：'`NVL(OD.RTXCANCELMARK,' ') = ' ' AND '`。

> 查询资料得知Oracle将空字符串即'&#160;&#160;'当成`NULL`处理。([参考链接](https://edgenhuang.iteye.com/blog/975567))    

若该订单未被取消，则`NVL(……)`返回' '，表达式变成`' ' = ' '`，恒成立，等于没写这句话，会留下未被取消的订单；若该订单被取消，则`NVL(……)`返回该订单的`RTXCANCAELMARK`（98），显然不等于' '，则被取消的订单会被从结果集中剔除。

- 子查询的结果集命其别名为`STATUS`，子查询语句如下：

```sql
SELECT
  o.code,
  t.description 
FROM
  ORDERSTATUSSETUP o,
  translationlist t
WHERE 
  t.code = o.code AND
  t.tblname='ORDERSTATUSSETUP' AND
  t.locale = 'zh' AND
  t.columnname ='DESCRIPTION';

```

子查询通过数据库中的数据字典`TRANSLATIONLIST`表将状态码与其描述关联，其中先检索表来梳理一下`TRANSLATIONLIST`的表结构：

```SQL
SELECT
  TBLNAME, JOINKEY1, JOINKEY2, COLUMNNAME, CODE,DESCRIPTION
FROM
  TRANSLATIONLIST
WHERE
  LOCALE = 'zh'
```

如上语句检索了`TRANSLATIONLIST`表中几个项，其中，`TBLNAME`指的是`table name`，存储了该行所属的表名；`CODE`是对应的状态码，`DESCRIPTION`是对应的描述文本（本地语言）。如订单的“外部取消”状态，其对应的`TBLNAME`值为“`ORDERSTATUSSETUP`”，指示了所属的表表名为“`ORDERSTATUSSETUP`”，`CODE`的值为98，`DESCRIPTION`是“外部取消”。  
检索结果集如下：

| TBLNAME  |   JOINKEY1    | JOINKEY2 | COLUMNNAME  | CODE | DESCRIPTION |
| :------: | :-----------: | :------: | :---------: | :--: | :---------: |
| CODELKUP | RTXORDTYPEMAP |   110    | DESCRIPTION | 110  |   BOX订单   |
| CODELKUP | RTXCANCELMARK |    98    | DESCRIPTION |  98  |  外部取消   |
| CODELKUP |    RTXPICK    |    B3    | DESCRIPTION |  B3  |  B3拣选面   |
| CODELKUP |    RTXPICK    |    B4    | DESCRIPTION |  B4  |  B4拣选面   |
|    ……    |      ……       |    ……    |     ……      |  ……  |     ……      |

子查询中的过滤条件中将`ORDERSTATUSSETUP`表与`TRANSLATIONLIST`表连接，连接条件为`TRANSLATIONLIST.CODE = ORDERSTATUSSETUP.CODE`，因为在`TRANSLATIONLIST`表中，单纯依靠`CODE`无法唯一标识每一行，`TRANSLATIONLIST`表采取多个字段组成复合主键，分别是：`TBLNAME`,`LOCALE`,`JOINKEY1`,`JOINKEY2`,`JOINKEY3`,`JOINKEY4`,`JOINKEY5`,`COLUMNNAME`.所以在子查询中需要借助于连接`ORDERSTATUSSETUP`来唯一确定状态码对应的描述。`ORDERSTATUSSETUP`表项不多，主要信息只有`CODE`和`DESCRIPTION`，其中`CODE`为表的主键，`DESCRIPTION`是英文文本描述。
子查询的结果集被命名为`STATUS`，结果如下（其中`CODE`源于`ORDERSTATUSSETUP`表，`DESCRIPTION`源于`TRANSLATIONLIST`表）：  

| CODE |     DESCRIPTION     |
| :--: | :-----------------: |
|  00  |     Empty Order     |
|  04  |      内部创建       |
|  04  |      内部创建       |
|  06  |       未分配        |
|  08  |      Converted      |
|  ……  |         ……          |
|  14  |      部分分配       |
|  15  |  部分分配/部分拣选  |
|  16  |  部分分配/部分运送  |
|  ……  |         ……          |
|  52  |      部分拣选       |
|  53  |  部分拣选/部分运送  |
|  55  |     拣货项完整      |
|  57  | 已全部拣货/部分运送 |
|  ……  |         ……          |

**注**：  

- `ORDERS`表中的`STATUS`字段表示了出货订单状态，下面是一些对应关系。   

| ORDERS.STATUS |     出货订单状态      |
| :-----------: | :-------------------: |
|      17       |        分配量         |
|      98       |       外部取消        |
|      99       |       内部取消        |
|      02       |       外部创建        |
|      04       |       内部创建        |
|      06       |        未分配         |
|      09       |        未开始         |
|      14       |       部分分配        |
|      15       |   部分分配/部分拣选   |
|      16       |   部分分配/部分运送   |
|      52       |       部分拣选        |
|      53       |   部分拣选/部分运送   |
|      22       |      已发放部分       |
|      25       | 已发放部分/已拣货部分 |
|      92       |       部分运送        |
|      57       |  已全部拣货/部分运送  |
|      55       |      拣货项完整       |
|      12       |       预分配量        |
|      29       |        已发放         |
|      95       |     出货全部完成      |

- `ORDERS`表的`TYPE`字段表示了订单类型，对应关系如下。  

| ORDERS.TYPE |  出货订单类型   |
| :---------: | :-------------: |
|     87      |     BOX订单     |
|     86      |     内淘宝      |
|     60      |  单款下架质检   |
|     82      | 拣货装箱单-唯品 |
|     84      | 拣货装箱单-国内 |
|     83      | 拣货装箱单-海外 |
|     120     | 拣货装箱单-退店 |
|     70      |    海外补单     |
|     140     |    海外订单     |
|     85      | 线上订单（B2C） |
|     81      |   采购退货单    |
|     80      |  预发WMS装箱单  |

- 此类代码与说明、描述的对应关系可以通过**INFOR_ENTERPRISE**内菜单栏  **管理→转换→代码转换**  进入查看。

### Example 02  

```SQL
SELECT
  O.ORDERKEY AS 订单号,
  S.DESCRIPTION AS 状态,
  TO_CHAR(O.ORDERDATE+8/24,'YYYY-MM-DD') AS 订单日期,
  TO_CHAR(O.ADDDATE+8/24,'YYYY-MM-DD') AS 下传日期,
  SUM(ORIGINALQTY) AS 数量 
FROM 
  ORDERS O,
  ORDERDETAIL OD,
  (SELECT 
    O.CODE,
    T.DESCRIPTION 
  FROM 
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T 
  WHERE 
    T.CODE = O.CODE AND 
    T.TBLNAME='ORDERSTATUSSETUP' AND 
    T.LOCALE = 'ZH' AND 
    T.COLUMNNAME ='DESCRIPTION'
    ) S 
WHERE 
  O.TYPE ='85' AND 
  O.ORDERKEY= OD.ORDERKEY AND 
  O.STATUS =S.CODE AND 
  TO_CHAR(O.ORDERDATE+8/24,'YYYY-MM-DD') >'2019-06-28' AND 
  TO_CHAR(O.ORDERDATE+8/24,'YYYY-MM-DD') <'2019-07-22'
GROUP BY 
  O.ORDERKEY,
  S.DESCRIPTION,
  O.ORDERDATE,
  O.ADDDATE
```

> Notes:
> 服务器设定为Greenwich时间,中国北京时间（东八区）与其换算关系为
> 北京时间 = GMT + 8(小时)
> 故数据库存储中数据项'2019-07-23 16:00:00'即是北京时间'2019-07-24 00:00:00'

### Example 03  

```SQL
--所有存储位的库存数据
SELECT 
  LLI.SKU,
  LLI.LOC,
  LLI.QTY 
FROM 
  LOTXLOCXID LLI,
  LOC L
WHERE 
  LLI.LOC = L.LOC AND 
  L.LOCATIONTYPE = 'PZBOXSTORE' AND 
  LLI.QTY > 0
```

```SQL
--所有零拣位的库存数据
SELECT 
  LLI.SKU,
  LLI.LOC,
  LLI.QTY 
FROM 
  LOTXLOCXID LLI,
  LOC L
WHERE 
  LLI.LOC = L.LOC AND 
  L.LOCATIONTYPE = 'PICK' AND 
  LLI.QTY > 0
```

### Example 04

```SQL
--各订单类型未完成清单
SELECT 
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间, 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间, 
  STATUS.DESCRIPTION 状态, 
  ORDERTYPE.DESCRIPTION 订单类型，
  COUNT(OD.ORDERKEY) 单数, 
  SUM(OD.TOTALQTY) 件数, 
  OD.RTXSHOPNAME
FROM 
  ORDERS OD, 
  (SELECT
    DISTINCT OD.TYPE,
    CL.DESCRIPTION
  FROM
    ORDERS OD,
    CODELKUP CL
  WHERE
    OD.TYPE = CL.CODE AND
    CL.LISTNAME = 'ORDERTYPE'
  ) ORDERTYPE,
  (SELECT
    O.CODE,
    T.DESCRIPTION 
  FROM
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE
    T.CODE = O.CODE AND
    T.TBLNAME='ORDERSTATUSSETUP' AND
    T.LOCALE = 'ZH' AND
    T.COLUMNNAME ='DESCRIPTION'
	) STATUS
WHERE 
  ORDERTYPE.TYPE = OD.TYPE AND
  STATUS.CODE = OD.STATUS AND 
  nvl(OD.rtxcancelmark,' ') = ' ' AND 
  OD.STATUS NOT IN ('98','55','95','99') AND
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= '2019-07-24' AND 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') >= '2019-06-28'
GROUP BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD'), 
  STATUS.DESCRIPTION, 
  ORDERTYPE.DESCRIPTION,
  OD.RTXSHOPNAME
ORDER BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD'),
  STATUS.DESCRIPTION,
  ORDERTYPE.DESCRIPTION，
  OD.RTXSHOPNAME
```

- 订单类型（`ORDERS`表中的`TYPE`字段）相对应的中文描述存储在`CODELKUP`表中，该表的主键为`LISTNAME`和`CODE`.  

### Example 05

```SQL
--7月22日B2C订单
SELECT 
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间, 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间, 
  STATUS.DESCRIPTION 状态, 
  COUNT(OD.ORDERKEY) 单数, 
  SUM(OD.TOTALQTY) 件数, 
  OD.RTXSHOPNAME
FROM
  ORDERS OD,
  (SELECT
      O.CODE,
      T.DESCRIPTION 
    FROM
      ORDERSTATUSSETUP O,
      TRANSLATIONLIST T
    WHERE
      T.CODE = O.CODE AND
      T.TBLNAME='ORDERSTATUSSETUP' AND
      T.LOCALE = 'ZH' AND
      T.COLUMNNAME ='DESCRIPTION'
  ) STATUS
WHERE 
  OD.TYPE = '85' AND
  STATUS.CODE = OD.STATUS AND 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') = '2019-07-22' 
GROUP BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD'), 
  STATUS.DESCRIPTION, 
  OD.RTXSHOPNAME
ORDER BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD'),
  STATUS.DESCRIPTION;
```

```SQL
--7月22日所有B2C订单详情
SELECT 
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间, 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间, 
  STATUS.DESCRIPTION 状态, 
  OD.EXTERNORDERKEY, 
  OD.TOTALQTY, 
  OD.RTXSHOPNAME
FROM 
  ORDERS OD, 
  (SELECT 
    O.CODE,
    T.DESCRIPTION 
  FROM 
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE 
    T.CODE = O.CODE AND 
    T.TBLNAME='ORDERSTATUSSETUP' AND 
    T.LOCALE = 'ZH' AND 
    T.COLUMNNAME ='DESCRIPTION'
  ) STATUS
WHERE 
  (OD.TYPE = '85' ) AND 
  STATUS.CODE = OD.STATUS AND 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') = '2019-07-22'
ORDER BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD');
```

### Example 06

```SQL
SELECT 
  O.ORDERKEY,
  S.DESCRIPTION,
  O.EXTERNORDERKEY,
  C.DESCRIPTION 订单类型,
  TO_CHAR(O.ORDERDATE+8/24,'YYYY-MM-DD') 订单日期,
  TO_CHAR(O.ADDDATE+8/24,'YYYY-MM-DD') 接单日期,--COUNT(DISTINCT O.ORDERKEY) 未发运订单数,
  SUM(OD.OPENQTY) 未结数量
FROM 
  ORDERS O,
  ORDERDETAIL OD,
  CODELKUP C,
  (SELECT 
    O.CODE,
    T.DESCRIPTION 
  FROM 
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE 
    T.CODE = O.CODE AND 
    T.TBLNAME='ORDERSTATUSSETUP' AND 
    T.LOCALE = 'ZH' AND 
    T.COLUMNNAME ='DESCRIPTION'
  )S 
WHERE 
  S.CODE = O.STATUS AND 
  O.TYPE ='85' AND 
  O.ORDERKEY=OD.ORDERKEY AND 
  C.CODE=O.TYPE AND 
  C.LISTNAME='ORDERTYPE' AND 
  NVL(O.RTXCANCELMARK,' ' ) = ' ' AND 
  O.STATUS < '95'  
GROUP BY 
  O.ORDERKEY,
  S.DESCRIPTION,
  O.EXTERNORDERKEY,
  C.DESCRIPTION,
  TO_CHAR(O.ORDERDATE+8/24,'YYYY-MM-DD'),
  TO_CHAR(O.ADDDATE+8/24,'YYYY-MM-DD')
ORDER BY 
  TO_CHAR(O.ORDERDATE+8/24,'YYYY-MM-DD'),
  C.DESCRIPTION;
```

### Example 07

```SQL
--6月25日到7月24日所有的B2C订单详情
SELECT 
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间, 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间, 
  STATUS.DESCRIPTION 状态, 
  OD.RTXCANCELMARK 取消标记，
  OD.ORDERKEY 订单号，
  OD.EXTERNORDERKEY 外部单号, 
  OD.TOTALQTY 总量, 
  OD.RTXSHOPNAME 店名
FROM 
  ORDERS OD, 
  (SELECT 
    O.CODE,
    T.DESCRIPTION 
  FROM 
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE 
    T.CODE = O.CODE AND 
    T.TBLNAME='ORDERSTATUSSETUP' AND 
    T.LOCALE = 'ZH' AND 
    T.COLUMNNAME ='DESCRIPTION'
  ) STATUS
WHERE 
  (OD.TYPE = '85' ) AND 
  NVL(OD.RTXCANCELMARK,' ') = ' ' AND
  STATUS.CODE = OD.STATUS AND 
  OD.STATUS NOT IN ('98','99','95') AND
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') >= '2019-06-25' AND 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= '2019-07-24' 
ORDER BY 
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'), 
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD')，
  STATUS.DESCRIPTION, 
  OD.ORDERKEY，
  OD.RTXCANCELMARK,
  OD.EXTERNORDERKEY, 
  OD.RTXSHOPNAME;
```

### Example 08

```SQL
--B2C/B2B剩余量
SELECT
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间,
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间,
  STATUS.DESCRIPTION 状态,
  --OD.RTXCANCELMARK 取消标记，
  OD.ORDERKEY 订单号，
  SUBSTR(OD.EXTERNORDERKEY,3) 外部单号, --去除开头的'OL'
  OD.TOTALQTY 总量,
  DECODE(NVL(OD.RTXSHOPNAME, ' '), ' ', OD.C_COMPANY, OD.RTXSHOPNAME) 店铺或收货人名称
FROM
  ORDERS OD,
  (SELECT
    O.CODE,
    T.DESCRIPTION
  FROM
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE
    T.CODE = O.CODE AND
    T.TBLNAME='ORDERSTATUSSETUP' AND
    T.LOCALE = 'zh' AND
    T.COLUMNNAME ='DESCRIPTION'
  ) STATUS
WHERE
  (OD.TYPE = '85' ) AND -- 84 为 拣货装箱单-国内，85 为 线上订单（B2C）
  NVL(OD.RTXCANCELMARK,' ') = ' ' AND  
  STATUS.CODE = OD.STATUS AND
  OD.STATUS NOT IN ('98','99','95') AND
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') >= '2019-06-25' AND
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= '2019-07-28'
  --TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= TO_CHAR(SYSDATE,'YYYY-MM-DD')
ORDER BY
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'),
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD')，
  STATUS.DESCRIPTION,
  OD.ORDERKEY，
  --OD.RTXCANCELMARK,
  SUBSTR(OD.EXTERNORDERKEY,3),
  DECODE(NVL(OD.RTXSHOPNAME, ' '), ' ', OD.C_COMPANY, OD.RTXSHOPNAME);
```

- `substr()`函数
  `substr()`函数有如下两种格式：
  - `substr(string string, int a, int b);`
    - string 需要截取的字符串
    - a 截取字符串的开始位置（注：当a等于0或1时，都是从第一位开始截取）
    - b 要截取的字符串的长度
  - `substr(string string, int a);`
    - string 需要截取的字符串
    - 可以理解为从第a个字符开始截取后面所有的字符串。
- **实例解析**

```SQL
	select substr('HelloWorld',0,3) value from dual; 
	//返回结果：Hel，截取从“H”开始3个字符
	select substr('HelloWorld',1,3) value from dual; 
	//返回结果：Hel，截取从“H”开始3个字符
	select substr('HelloWorld',2,3) value from dual; 
	//返回结果：ell，截取从“e”开始3个字符
	select substr('HelloWorld',0,100) value from dual; 
	//返回结果：HelloWorld，100虽然超出预处理的字符串最长度，但不会影响返回结果，系统按预处理字符串最大数量返回。
	select substr('HelloWorld',5,3) value from dual; 
	//返回结果：oWo
	select substr('Hello World',5,3) value from dual; 
	//返回结果：o W (中间的空格也算一个字符串，结果是：o空格W)
	select substr('HelloWorld',-1,3) value from dual; 
	//返回结果：d （从后面倒数第一位开始往后取1个字符，而不是3个。原因：下面红色 第三个注解）
	select substr('HelloWorld',-2,3) value from dual; 
	//返回结果：ld （从后面倒数第二位开始往后取2个字符，而不是3个。原因：下面红色 第三个注解）
	select substr('HelloWorld',-3,3) value from dual; 
	//返回结果：rld （从后面倒数第三位开始往后取3个字符）
	select substr('HelloWorld',-4,3) value from dual; 
	//返回结果：orl （从后面倒数第四位开始往后取3个字符）
	select substr('HelloWorld',0) value from dual;  
	//返回结果：HelloWorld，截取所有字符
	select substr('HelloWorld',1) value from dual;  
	//返回结果：HelloWorld，截取所有字符
	select substr('HelloWorld',2) value from dual;  
	//返回结果：elloWorld，截取从“e”开始之后所有字符
	select substr('HelloWorld',3) value from dual;  
	//返回结果：lloWorld，截取从“l”开始之后所有字符
	select substr('HelloWorld',-1) value from dual;  
	//返回结果：d，从最后一个“d”开始 往回截取1个字符
	select substr('HelloWorld',-2) value from dual;  
	//返回结果：ld，从最后一个“d”开始 往回截取2个字符
	select substr('HelloWorld',-3) value from dual;  
	//返回结果：rld，从最后一个“d”开始 往回截取3个字符
```

### Example 09

```SQL
SELECT 
  SKU 货品SKU,
  LOC 库位,
  FIRSTNO 一级分理编号,
  SECONDNO 二级分理编号,
  DECODE(STATUS,'0','未下发','1','已下发') 状态
FROM 
  RTX_SORTNO
ORDER BY
  SKU;
```

- `RTX_SORTNO`是一个视图。

### Example 10

```SQL
--按SKU合计数量
SELECT 
  SKU,
  SUM(QTY) AS 合计
FROM 
  LOTXLOCXID T 
WHERE 
  QTY > 0 AND 
  STATUS = 'OK'
GROUP BY
  SKU
ORDER BY 
  SKU;
```

```SQL
--按款式合计数量
SELECT 
  S.STYLE 款式,
  SUM(LLI.QTY) AS 合计数量
FROM 
  LOTXLOCXID LLI,
  SKU S
WHERE 
  LLI.SKU = S.SKU AND
  LLI.QTY > 0 AND 
  LLI.STATUS = 'OK'
GROUP BY
  S.STYLE
ORDER BY 
  S.STYLE;
```

### Example 11

```SQL
SELECT 
  S.STYLE 款号,
  MT.SKU 货品,
  MT.STORERKEY 货主,
  MT.FROMLOC 自库位,
  MT.TOLOC 至库位,
  MT.LOT 批次,
  MT.QTY 返架数量,
  DECODE(MT.STATUS,'10','扫描中','已返架') 状态,
  OE.FULLY_QUALIFIED_ID 用户,
  TO_CHAR(MT.EDITDATE+8/24,'YYYY-MM-DD HH24:MI:SS') 操作时间 
FROM 
  MANYMOVE_TBL MT,
  OPER.E_SSO_USER OE,
  SKU S 
WHERE 
  UPPER(OE.SSO_USER_NAME)=UPPER(MT.EDITWHO) AND 
  MT.SKU=S.SKU 
ORDER BY 
  TO_CHAR(MT.EDITDATE+8/24,'YYYY-MM-DD HH24:MI:SS') DESC;
```

- 用户的登录ID和姓名存储在`OPER.E_SSO_USER`表中，其中，`SSO_USER_NAME`为登录账户，`FULLY_QUALIFIED_ID`为用户全名。
- 比较的时候要注意，PL/SQL的字符串比较是大小写敏感的，因此在多表连接比较操作人ID时应当注意全部转化为大写再进行比较。如如上过滤条件中`UPPER(OE.SSO_USER_NAME)=UPPER(MT.EDITWHO)`这样写。

### Example 12

```SQL
--库存事务页面信息对应ITRN表
--统计一个月前至今的库存转移量（THFLLOC→状态为OK的零拣位）
(
SELECT 
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD')||' 小计' 日期,
  NULL 货品SKU,
  NULL 批号LOT,
  NULL 自库位,
  NULL 至库位,
  NULL 用户,
  SUM(IT.QTY) 数量
FROM
  ITRN IT,
  LOC L
WHERE
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') >= TO_CHAR(ADD_MONTHS(TRUNC(SYSDATE),-1),'YYYY-MM-DD') AND
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') <= TO_CHAR(SYSDATE,'YYYY-MM-DD') AND
  IT.FROMLOC = 'THFLLOC' AND
  IT.TOLOC = L.LOC AND
  L.LOCATIONTYPE = 'PICK' AND
  L.STATUS = 'OK'
GROUP BY TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD')
)
UNION
SELECT * FROM
(
SELECT
  --TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD HH24:MI:SS') 日期,
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') 日期,
  IT.SKU 货品SKU,
  IT.LOT 批号LOT,
  IT.FROMLOC 自库位,
  IT.TOLOC 至库位,
  OE.FULLY_QUALIFIED_ID 用户,
  IT.QTY 数量
FROM
  ITRN IT,
  LOC L,
  OPER.E_SSO_USER OE
WHERE
  --从一个月前开始统计
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') >= TO_CHAR(ADD_MONTHS(TRUNC(SYSDATE),-1),'YYYY-MM-DD') AND
  --到系统时间
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') <= TO_CHAR(SYSDATE,'YYYY-MM-DD') AND
  IT.FROMLOC = 'THFLLOC' AND
  IT.TOLOC = L.LOC AND
  L.LOCATIONTYPE = 'PICK' AND
  L.STATUS = 'OK' AND
  UPPER(IT.ADDWHO) = UPPER(OE.SSO_USER_NAME)
ORDER BY 
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD HH24:MI:SS'),
  --TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD'),
  IT.SKU,
  IT.LOT,
  IT.FROMLOC,
  IT.TOLOC,
  OE.FULLY_QUALIFIED_ID,
  IT.QTY
)
UNION
(
SELECT 
  '合计' 日期,
  NULL 货品SKU,
  NULL 批号LOT,
  NULL 自库位,
  NULL 至库位,
  NULL 用户,
  SUM(IT.QTY) 数量
FROM
  ITRN IT,
  LOC L
WHERE
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') >= TO_CHAR(ADD_MONTHS(TRUNC(SYSDATE),-1),'YYYY-MM-DD') AND
  TO_CHAR(IT.EFFECTIVEDATE+8/24, 'YYYY-MM-DD') <= TO_CHAR(SYSDATE,'YYYY-MM-DD') AND
  IT.FROMLOC = 'THFLLOC' AND
  IT.TOLOC = L.LOC AND
  L.LOCATIONTYPE = 'PICK' AND
  L.STATUS = 'OK'
)
```

- `union`用的比较多，`union all`是直接连接，取到的是所有值，记录可能有重复。   `union` 是取唯一值，记录没有重复。
- `UNION`的语法如下：

```SQL
	[SQL Statement 1]
	UNION
	[SQL Statement 2]  
```

- `UNION ALL`的语法如下：

```SQL
	[SQL Statement 1]
	UNION ALL
	[SQL Statement 2]
```

- 对重复结果的处理：`UNION`在进行表链接后会筛选掉重复的记录，`Union All`不会去除重复记录。对排序的处理：`Union`将会按照字段的顺序进行排序；`UNION ALL`只是简单的将两个结果合并后就返回。

- 多语句`UNION`时，报错未正确结束。原因是在每个语句尾部都有`ORDER BY`子句，而规定在Oracle PL/SQL中，`ORDER BY`子句必须是`SELECT`语句的最后一个语句，且一个`SELECT`语句中只允许出现一个`ORDER BY`语句，同时，`ORDER BY`子句必须位于**整个**`SELECT`语句的最末尾。

- 如上问题的解决方法： 

  1. 将结果集作为一张临时表，再对其查询、排序。

  ```SQL
  	SELECT * FROM (…… UNION ……) ORDER BY ………
  ```

  2. 排序后再合并。

  ```SQL
  	SELECT * FROM (SELECT …… FROM …… ORDER BY ……)
  	UNION
  	SELECT * FROM (SELECT …… FROM …… ORDER BY ……)
  ```

  3. ORDER BY + 字段在结果集中的序号。

  ```SQL
  	SELECT ……
  	UNION
  	SELECT ……
  	ORDER BY 1,2
  ```

### Example 13

```SQL
--库存中所有零拣位状态为OK的库存
SELECT 
  LLI.SKU,
  SUM(LLI.QTY) AS 合计
FROM 
  LOTXLOCXID LLI,
  LOC L 
WHERE 
  LLI.QTY > 0 AND 
  LLI.STATUS = 'OK' AND
  LLI.LOC = L.LOC AND
  L.LOCATIONTYPE = 'PICK' 
GROUP BY
  LLI.SKU
ORDER BY 
  LLI.SKU;
```

### Example 14  

```SQL
--一段时间内的订单信息
SELECT
  DISTINCT TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') 订单时间,
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD') 下传时间,
  STATUS.DESCRIPTION 状态,
  --OD.RTXCANCELMARK 取消标记，
  OD.ORDERKEY 订单号，
  OD.EXTERNORDERKEY 外部单号,
  SUBSTR(OD.EXTERNORDERKEY,3) 截取的外部单号, --去除开头的'OL'/'OW'
  OD.CARRIERNAME 承运人,
  OD.RTXEXPRESSKEY 快递单号,
  OD.TOTALQTY 总量,
  DECODE(NVL(OD.RTXSHOPNAME, ' '), ' ', OD.C_COMPANY, OD.RTXSHOPNAME) 店铺或收货人名称
FROM
  ORDERS OD,
  (SELECT
    O.CODE,
    T.DESCRIPTION
  FROM
    ORDERSTATUSSETUP O,
    TRANSLATIONLIST T
  WHERE
    T.CODE = O.CODE AND
    T.TBLNAME='ORDERSTATUSSETUP' AND
    T.LOCALE = 'zh' AND
    T.COLUMNNAME ='DESCRIPTION'
  ) STATUS
WHERE
  OD.TYPE IN ('85','86') AND -- 84 为 拣货装箱单-国内，85 为 线上订单（B2C）
  NVL(OD.RTXCANCELMARK,' ') = ' ' AND
  STATUS.CODE = OD.STATUS AND
  --OD.STATUS NOT IN ('98','99','95') AND
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') >= '2019-06-01' AND
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= '2019-07-31'
  --TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD') <= TO_CHAR(SYSDATE,'YYYY-MM-DD')
ORDER BY
  TO_CHAR(OD.ORDERDATE+8/24, 'YYYY-MM-DD'),
  TO_CHAR(OD.ADDDATE+8/24, 'YYYY-MM-DD'),
  STATUS.DESCRIPTION,
  OD.ORDERKEY，
  --OD.RTXCANCELMARK,
  SUBSTR(OD.EXTERNORDERKEY,3),
  DECODE(NVL(OD.RTXSHOPNAME, ' '), ' ', OD.C_COMPANY, OD.RTXSHOPNAME);
```

## 小工具

### 几条实用的语句

```SQL
--在所有表中按字段名搜索字段存在于哪些表中
SELECT 
  *
FROM 
  USER_TAB_COLUMNS 
WHERE
  COLUMN_NAME = 'ORDERKEY'
ORDER BY
  TABLE_NAME; 
```

```sql
--查找指定表的主键
SELECT 
  CU.* 
FROM 
  USER_CONS_COLUMNS CU, 
  USER_CONSTRAINTS AU 
WHERE 
  CU.CONSTRAINT_NAME = AU.CONSTRAINT_NAME AND
  AU.CONSTRAINT_TYPE = 'P' AND
  CU.TABLE_NAME = 'SKU';
```

```sql
--查找指定表中所有的列名
----可以加上过滤条件以达到在表中搜索字段的目的
SELECT 
  T.COLUMN_NAME 
FROM 
  USER_TAB_COLUMNS T 
WHERE 
  T.TABLE_NAME = 'SKU' --AND
  --T.COLUMN_NAME LIKE '%STATUS%' 
ORDER BY
  T.COLUMN_NAME;
```

|          日期说明           |         Oracle语句（假设现在是2018-11-28 11:11:11）          |      返回日期       |
| :-------------------------: | :----------------------------------------------------------: | :-----------------: |
|         当月第一天          |            `select trunc(sysdate,'mm') from dual`            |      2018-11-1      |
|         当年第一天          |            `select trunc(sysdate,'yy') from dual`            |      2018-1-1       |
|         当前年月日          |            `select trunc(sysdate,'dd') from dual`            |     2018-11-28      |
| 当前星期的第一天 （星期天） |            `select trunc(sysdate,'d') from dual`             |     2018-11-28      |
|          当前日期           |              `select trunc(sysdate) from dual`               |     2018-11-28      |
|   当前时间（准确到小时）    |            `select trunc(sysdate,'hh') from dual`            | 2018-11-28 11:00:00 |
|   当前时间（准确到分钟）    | `select to_char(trunc(sysdate,'mi'),'yyyy-MM-dd HH:mm:ss') from dual` | 2018-11-28 11:11:00 |
|        前一天的日期         |             `select TRUNC(SYSDATE-1) from dual`              |     2018-11-27      |
|       前一个月的日期        |       `select add_months(trunc(sysdate),-1) from dual`       |     2018-10-28      |
|       后一个月的日期        |       `select add_months(trunc(sysdate),1) from dual`        |     2018-12-28      |
|        本月最后一天         |  `select to_char(last_day(sysdate),'yyyy-mm-dd') from dual`  |     2018-11-30      |

### 参考链接和常用

1. [Oracle中union all、union 和order by一起使用的解决方法](https://blog.csdn.net/luzaijx/article/details/83310472)

2. [oracle 根据字段名查找表](https://blog.csdn.net/u012382571/article/details/49074505)

3. [Oracle获取日期大全（当月的第一天/后一天/上一天/最后一天/上个月这一天）](https://blog.csdn.net/moshowgame/article/details/84581821)

4. [ORACLE制作表时的“小计”和“合计” （ROLLUP）](https://blog.csdn.net/shcqupc/article/details/51037515)

5. [PL/SQL的TO_CHAR（）与TO_DATE（）](https://www.cnblogs.com/lmfeng/archive/2011/08/16/2140737.html)

6. [oracle trunc()函数的用法——日期、数字](https://blog.csdn.net/qq_30934019/article/details/80611489)

7. [oracle中实现截取字符串（substr）、查找字符串位置（instr）、替换字符串（replace）](https://blog.csdn.net/housonglin1213/article/details/50344895)

8. [【ORACLE】Oracle提高篇之DECODE](https://blog.csdn.net/sdut406/article/details/82795585)

9. [解决Oracle数据库ORA-28001](https://my.oschina.net/NamiZone/blog/1840232)

10. [【SQL】两个带order by查询进行union all报ORA-00933错误的解决方法](https://blog.csdn.net/zhx624/article/details/20373785)

11. [Oracle中查看所有表和字段以及表注释.字段注释](https://www.cnblogs.com/xusir/p/3214714.html)

12. [Oracle中的rowid](https://www.cnblogs.com/xqzt/p/4449184.html)

13. [数据库中乐观锁、悲观锁、共享锁和排它锁的理解](https://www.liangzl.com/get-article-detail-1144.html)

14. [查询oracle表的信息（表，字段，约束，索引）](https://www.cnblogs.com/furenjian/articles/2907688.html)

15.  [《Oracle PL/SQL必知必会》](https://book.douban.com/subject/27004312/)

<div style="text-align: right"> 柳明 </div>
<div style="text-align: right"> 2019.08.03 </div>
