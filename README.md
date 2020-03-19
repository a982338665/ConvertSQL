
###存储过程初版来源：https://www.cnblogs.com/xinysu/p/6992415.html?utm_source=itdadao&utm_medium=referral

**1.安装mssql：**

    npm install mssql
    
**2.官网：**

    https://www.npmjs.com/package/mssql
    
**3.类型转换PROC：**

    1.位置：D:\git-20191029\ConvertSQL\proc
    2.执行：exec p_tb_mssqltomysql 'tbtest' -- tbtest为表名
   
**4.使用convert2.sql碰到的问题:**
    
    1.问题：第 2行id=7879295【v_Production_Line_Proc_Units  mysql表创建问题！ER_TOO_BIG_ROWSIZE: Row size too large. 
            The maximum row size for the used table type, not counting BLOBs, 
            is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
      原因：MySQL对于每行存放数据的字节数总和是有限制的，最大字节数为65535，即64k。而这个限制条件，是不包含类型为text和blob的。
            如果存放的时utf数据，那么64k大概可以存放21845个 utf8字符。
            对于大文本字段或大字节字段建议使用text和blob，为了提高查询效率，大文本/大字节字段需要单独出一个子表存放
      解决：convert3.sql,将varchar>255的都换为text类型
      
**5.关于代码中无法建表的原因：**
    
    1.第 459行id=1934734045【Timed_Event_Details_NPT  暂不支持！【convert4.sql】
        ·执行：exec p_tb_mssqltomysql 'Timed_Event_Details_NPT_0001';
        ·获取创建语句：CREATE TABLE Timed_Event_Details_NPT_0001(TEDet_Id int  not null  auto_increment  
                                                             ,Action_Comment_Id int  null  
                                                             ...
                                                             );
        ·报错：Incorrect table definition; there can be only one auto column and it must be defined as a key
        ·解决：添加为主键自增，或者放弃auto_increment
        ·见convert4.sql
    2.第 191行id=759829919【SDK_V_PAUserParameterValue  mysql表创建问题！ER_PARSE_ERROR: You have an error in your SQL syntax; check the manual that corresponds to your 
        MySQL server version for the right syntax to use near 'MaxValue int  null
        ·执行：exec p_tb_mssqltomysql 'SDK_V_PAUserParameterValue_0001';
        ·获取建表语句：
            CREATE TABLE SDK_V_PAUserParameterValue_001(ParameterId int  not null  
                                                                 ,Value text  not null  
                                                                 ,MinValue int  null  
                                                                 ,MaxValue int  null  
                                                                 ...
                                                                 );
        ·问题：MaxValue关键字，作为字段时要写为：`MaxValue`
        ·解决：修改存储过程
        ·见convert5.sql
    3.'第 53行id=218744132【v_ERP_PrintOrder|建表语句转换问题|】null <br/>',
      ·查看视图内容：
            create view v_ERP_PrintOrder
            as
            select *from (
             select  * from openquery([10.40.1.1],'select * FROM cux_purchase_order_for_mes where  PO_STATUS = ''批准'' and SubStr(ITEM_NUM,1,2)=''33'' and QTY_ORDERED>QTY_RECEIVED'))a
            GO                                          
      ·执行：在 sys.servers 中找不到服务器 '10.40.1.1'。请验证指定的服务器名称是否正确。如果需要，请执行存储过程 sp_addlinkedserver 以将服务器添加到 sys.servers。
      ·问题：无法执行，即无法确定列字段，暂不解决
    4.'第 211行id=837174328【v_mm_MaterialCoastStatistics|建表语句转换问题|】null <br/>',
      ·问题：无法执行，视图列名无效，视图不正确
    
