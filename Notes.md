# SQL 学习笔记

![Oracle 图标](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1564591842751&di=3cb0b9bc69df78d08816abe6101c7fc1&imgtype=0&src=http%3A%2F%2Fwww.soomal.com%2Fimages%2Fdoc%2F20110713%2F00012096.jpg "Oracle logo")

###Example 01

```SQL

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

            T.COLUMNNAME ='DESCRIPTION') STATUS

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

*&emsp;&emsp; `ORDERS`表中，订单的取消标记`RTXCANCELMARK`和`RTXCANCELBACK`两个项存储，其中与WMS出货单页面上的“取消标识”对应的是`RTXCANCELBACK`。`RTXCANCELMARK`的取值和`RTXCANCELBACK`取值如下：

| RTXCANCELMARK  | RTXCANCELBACK  |

|:--------------:|:--------------:|

| 98            | Y              |

|                | N              |

* &emsp;&emsp;某订单若被标记取消，该订单的`RTXCANCELMARK`字段为98。`RTXCANCELBACK`字段为Y表示YES即已取消；若未被标记取消，该订单的`RTXCANCELMARK`字段为一个空字符串即'&#160;&#160;'，搜索时可以用'&#160;&#160;'作为关键字搜索。

* &emsp;&emsp;`nvl()`作为Oracle/PLSQL中的一个函数，其格式为：`nvl(string1, replace_with)`，它的功能如下：如果string1为`NULL`，则NVL函数返回`replace_with`的值，否则返回`string1`的值。

&emsp;&emsp;在本SQL语句段的过滤条件中，这一小句这样写：'`NVL(OD.RTXCANCELMARK,' ') = ' ' AND '`。

>查询资料得知Oracle将空字符串即'&#160;&#160;'当成`NULL`处理。([参考链接](https://edgenhuang.iteye.com/blog/975567))   

&emsp;&emsp;若该订单未被取消，则`NVL(……)`返回' '，表达式变成`' ' = ' '`，恒成立，等于没写这句话，会留下未被取消的订单；若该订单被取消，则`NVL(……)`返回该订单的`RTXCANCAELMARK`（98），显然不等于' '，则被取消的订单会被从结果集中剔除。

* &emsp;&emsp;子查询的结果集命其别名为`STATUS`，子查询语句如下：

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

        t.columnname ='DESCRIPTION'

```

&emsp;&emsp;子查询通过数据库中的数据字典`TRANSLATIONLIST`表将状态码与其描述关联，其中先检索表来梳理

一下`TRANSLATIONLIST`的表结构：

```SQL

    SELECT

      TBLNAME, JOINKEY1, JOINKEY2, COLUMNNAME, CODE,DESCRIPTION

    FROM

      TRANSLATIONLIST

    WHERE

      LOCALE = 'zh'

``` 

&emsp;&emsp;如上语句检索了`TRANSLATIONLIST`表中几个项，其中，`TBLNAME`指的是`table name`，存储了该行所属的表名；`CODE`是对应的状态码，`DESCRIPTION`是对应的描述文本（本地语言）。如订单的“外部取消”状态，其对应的`TBLNAME`值为“`ORDERSTATUSSETUP`”，指示了所属的表表名为“`ORDERSTATUSSETUP`”，`CODE`的值为98，`DESCRIPTION`是“外部取消”。 

&emsp;&emsp;检索结果集如下：

| TBLNAME    | JOINKEY1      | JOINKEY2 | COLUMNNAME  | CODE  | DESCRIPTION |

|:-----------:|:-------------:|:--------:|:-----------:|:------:|:-----------:|

| CODELKUP    | RTXORDTYPEMAP | 110      | DESCRIPTION | 110    | BOX订单      |

| PUTAWAYZONE | DFZONE        | DFZONE  | DESCR      | DFZONE | 待发货区        |

| PUTAWAYZONE | JHZONE        | JHZONE  | DESCR      | JHZONE | 货品集货区      |

| CODELKUP    | RTXCANCELMARK | 98      | DESCRIPTION | 98    | 外部取消        |

| CODELKUP    | RTXPICK      | B3      | DESCRIPTION | B3    |  B3拣选面      |

| CODELKUP    | RTXPICK      | B4      | DESCRIPTION | B4    |  B4拣选面      |

| ……          | ……            | ……      | ……          | ……    | ……          |

&emsp;&emsp;子查询中的过滤条件中将`ORDERSTATUSSETUP`表与`TRANSLATIONLIST`表连接，连接条件为`TRANSLATIONLIST.CODE = ORDERSTATUSSETUP.CODE`，因为在`TRANSLATIONLIST`表中，单纯依靠`CODE`无法唯一标识每一行，`TRANSLATIONLIST`表采取多个字段组成复合主键，分别是：`TBLNAME`,`LOCALE`,`JOINKEY1`,`JOINKEY2`,`JOINKEY3`,`JOINKEY4`,`JOINKEY5`,`COLUMNNAME`.所以在子查询中需要借助于连接`ORDERSTATUSSETUP`来唯一确定状态码对应的描述。`ORDERSTATUSSETUP`表项不多，主要信息只有`CODE`和`DESCRIPTION`，其中`CODE`为表的主键，`DESCRIPTION`是英文文本描述。

&emsp;&emsp;子查询的结果集被命名为`STATUS`，结果如下（其中`CODE`源于`ORDERSTATUSSETUP`表，`DESCRIPTION`源于`TRANSLATIONLIST`表）：

| CODE | DESCRIPTION |

|:----:|:-----------:|

| 00  | Empty Order |

| 04  | 内部创建        |

| 04  | 内部创建        |

| 06  | 未分配        |

| 08  | Converted  |

| ……  | ……          |

| 14  | 部分分配        |

| 15  | 部分分配/部分拣选  |

| 16  | 部分分配/部分运送  |

| ……  | ……          |

| 52  | 部分拣选        |

| 53      | 部分拣选/部分运送  |

| 55  | 拣货项完整      |

| 57  | 已全部拣货/部分运送  |

| ……  | ……          |