[TOC]

#什么是liquibase

开源地址：https://github.com/liquibase/liquibase

官网地址：[http://www.liquibase.org/](http://www.liquibase.org/)

官方指导文档：[http://www.liquibase.org/documentation/index.html](http://www.liquibase.org/documentation/index.html)

     Liquibase是一个用于跟踪、管理和应用数据库变化的开源的数据库重构工具。它将所有数据库的变化（包括结构和数据）都保存在XML文件中，便于版本控制。

     Liquibase具备如下特性：
1. 不依赖于特定的数据库，目前支持包括Oracle/Sql Server/DB2/MySql/Sybase/PostgreSQL/Caché等12种数据库，这样在数据库的部署和升级环节可帮助应用系统支持多数据库。
2. 提供数据库比较功能，比较结果保存在XML中，基于该XML你可用Liquibase轻松部署或升级数据库。
3. 以XML存储数据库变化，其中以作者和ID唯一标识一个变化（ChangSet），支持数据库变化的合并，因此支持多开发人员同时工作。
4. 在数据库中保存数据库修改历史（DatabaseChangeHistory），在数据库升级时自动跳过已应用的变化（ChangSet）。
5. 提供变化应用的回滚功能，可按时间、数量或标签（tag）回滚已应用的变化。通过这种方式，开发人员可轻易的还原数据库在任何时间点的状态。
6. 可生成数据库修改文档（HTML格式）
7. 提供数据重构的独立的IDE和Eclipse插件

(简介摘自：[http://www.xuebuyuan.com/45332.html)](http://www.xuebuyuan.com/45332.html))

#liquibase官方文档解读

##Database Change Log文件

liquibase变更修改文件是一个根节点元素名为databaseChangeLog的文件。该文件记录了数据的添加、删除操作字段、初始化等变更操作。

databaseChangeLog元素内可以包含属性有：

1. logicalFilePath：表示databasechangelog唯一标识，默认应该是使用文件名做唯一标识的，官方文档建议当文件重命名或者移动的时候添加该参数用来告诉liquibase这个是个重命名或移动文件。（没有验证，等待测试～）

##databaseChangeLog子标签

###preConditions

执行changeLog的前置条件。preConditions标签位于databaseChangeLog下或者是changeSet标签下。（详细描述：http://www.liquibase.org/documentation/preconditions.html）适用场景：

1. Document what assumptions the writers of the changelog had when creating it.

2.Enforce that those assumptions are not violated by users running the changelog

3. 执行一些不可会退语句之前的数据检查，例如drop_table操作前语句检查。

4. 根据数据状态（数据库类型、操作用户账号信息等）操作哪些changesets需要执行，哪些changsets不执行。

###property

参数值，可以在设置一下通用的参数值，将使用测试较多的变量提出来使用。使用方法如下：

```

<databaseChangeLog

        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"

        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"

        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd

        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <property name="clob.type" value="clob" dbms="oracle"/>

    <property name="clob.type" value="longtext" dbms="mysql"/>

    <changeSet id="1" author="joe">

         <createTable tableName="${table.name}">

             <column name="id" type="int"/>

             <column name="${column1.name}" type="${clob.type}"/>

             <column name="${column2.name}" type="int"/>

         </createTable>

    </changeSet>

</databaseChangeLog>

```

###changeSet

需要执行的数据变化语句，该标签是liquibase很重要的一个配置项，管理着数据库变更的核心内容。每一个changeSet标签需要指定‘id’、‘author’和changlog文件classpath路径信息。id属性是一个唯一标识码，这个属性不代表执行changeset顺序，是一个字符串，可以没有数字。填写changeSet的时候如果不知道或者不想填写author，可以写一个简单的标识例如像“UNKNOW”。

当liquibase执行databaseChangeLog的时，按顺序读取changeSets。当运行时liquibase读取databaseChangeLog数据库查看每一个changeSet是否被执行过，如果执行过则跳过该changeSet，识别changeSet是根据id/author/filePath来识别的，如果配置了runAlways属性为true，该changeSet则会每次执行。当databaseChangeLog中每一条changeSet执行完成后liquibase会在databaseChangeLog数据库中添加一条记录。

liquibase尝试保证修改数据库事务操作，他会在执行在最后提交修改，在出现错误时尝试回滚，一些数据库使用了自动提交事物可能会导致错误，因此建议每个changeSet标签仅表示一个变更，像插入数据这样一组数据插入非自动提交的变更可以写在同一个changeSet里面。

changeSet中属性包括以下几个：

1. id：字母数字标志符号

2. author：changeSet创建者信息

3. dbms：数据库类型

4. runAlways：是否每次都执行该chageSet

5. runOnChange：第一次和当changeSet发生修改的时候执行该changeSet

6. context：Executes the change if the particular context was passed at runtime. Any string can be used for the context name and they are checked case-insensitively.

7. runInTransaction：如果可以的话，该changeSet保证事务执行变更，默认参数是true，谨慎操作该参数，如果该参数修改成false，当执行数据库变更时候出现错误导致停止，datbaseChangeLog数据库将会变成无效的状态发生错误！

8. failOnError：Should the migration fail if an error occurs while executing the changeSet?

changeSet可供使用的子标签：

1. comment：changeSet的描述

2. preConditions：同上面的描述

3. rollback：发生回滚时候需要执行的语句

4. validCheckSum：List checksums which are considered valid for this changeSet, regardless of what is stored in the database. Used primarily when you need to change a changeSet and don't want errors thrown on databases on which it has already run (not a recommended procedure). Since 1.7

###incloud

添加一个额外的文件描述changeLog。将可能复杂很长的一个changeLog描述文件分割成好几个部分。可用户属性有：

1. file：需要导入的文件名字（必需）

2. relativeToChangelogFile：文件相对于root changeLog文件路径，默认是false，表示默认使用classpath路径查找文件。

当前存在的问题和一下注意：没有解决循环引用文件问题，当你引用一个文件两次不用担心changeSet将会执行两次，因为当第二次执行的时候databasechangelog会发现changeSet已经执行过了会忽这个变更。注意！不要循环引用会导致严重问题。

##一个简单的示例

这个例子里面使用了property、基本的创建表、include、comment等上面说过的配置信息

```

<?xml version="1.0" encoding="utf-8"?>

<databaseChangeLog

        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"

        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog

        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">

    <include file="classpath:liquibase/change_log/init-schema.xml" relativeToChangelogFile="false"/>

</databaseChangeLog>

```

```

<?xml version="1.0" encoding="utf-8"?>

<databaseChangeLog

        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"

        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog

        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">

    <property name="autoIncrement" value="true" dbms="mysql"/>

    <changeSet id="init-schema" author="yc" >

        <comment>init schema</comment>

        <createTable tableName="user">

            <column name="id" type="bigint" autoIncrement="${autoIncrement}">

                <constraints primaryKey="true" nullable="false"/>

            </column>

            <column name="name" type="varchar(255)">

                <constraints  nullable="false"/>

            </column>

            <column name="age" type="int(11))">

                <constraints  nullable="false"/>

            </column>

            <column name="email" type="varchar(255)">

                <constraints  nullable="false"/>

            </column>

        </createTable>

        <modifySql dbms="mysql">

            <append value="ENGINE=INNODB DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci"/>

        </modifySql>

    </changeSet>

</databaseChangeLog>

```