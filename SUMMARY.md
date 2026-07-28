# README 页面

* [LiteDB: 设计与实践](README.md)

# 关于 LiteDB

* [关于 LiteDB](about/README.md)

    * [设计目标](about/design_goals/README.md)

    * [核心特性](about/core_features/README.md)

    * [应用场景](about/use_cases/README.md)

# 开始使用

* [开始使用](getting_started/README.md)

    * [下载源码](getting_started/installation/README.md)

    * [编译源码](getting_started/building/README.md)

    * [快速使用](getting_started/quick_start/README.md)

# SQL 语句

* [SQL 语句](sql/README.md)

    * [数据类型](sql/data_types/README.md)

        * [数据类型总览](sql/data_types/overview/README.md)

        * [VARCHAR(n)](sql/data_types/varchar/README.md)

        * [BOOLEAN](sql/data_types/boolean/README.md)

        * [VECTOR(n)](sql/data_types/vector/README.md)

    * [DDL 语句](sql/ddl/README.md)

        * [数据库管理](sql/ddl/database/README.md)

            * [USE](sql/ddl/database/use/README.md)

            * [CREATE DATABASE](sql/ddl/database/create_database/README.md)

            * [DROP DATABASE](sql/ddl/database/drop_database/README.md)

            * [SHOW DATABASES](sql/ddl/database/show_databases/README.md)

        * [集合管理](sql/ddl/collection/README.md)

            * [CREATE COLLECTION](sql/ddl/collection/create_collection/README.md)

            * [DROP COLLECTION](sql/ddl/collection/drop_collection/README.md)

            * [SHOW COLLECTIONS](sql/ddl/collection/show_collections/README.md)

            * [DESCRIBE](sql/ddl/collection/describe/README.md)

        * [标量索引](sql/ddl/scalar_index/README.md)

            * [CREATE INDEX](sql/ddl/scalar_index/create_index/README.md)

            * [DROP INDEX](sql/ddl/scalar_index/drop_index/README.md)

            * [SHOW INDEXES](sql/ddl/scalar_index/show_indexes/README.md)

        * [向量索引](sql/ddl/vector_index/README.md)

            * [CREATE VINDEX](sql/ddl/vector_index/create_vindex/README.md)

            * [DROP VINDEX](sql/ddl/vector_index/drop_vindex/README.md)

            * [SHOW VINDEXES](sql/ddl/vector_index/show_vindexes/README.md)

    * [DML 语句](sql/dml/README.md)

        * [INSERT](sql/dml/insert/README.md)

        * [UPDATE](sql/dml/update/README.md)

        * [DELETE](sql/dml/delete/README.md)

    * [DQL 语句](sql/dql/README.md)

        * [SELECT](sql/dql/select/README.md)

    * [表达式](sql/expressions/README.md)

# 架构与设计

* [架构与设计](architecture_and_design/README.md)

    * [整体架构](architecture_and_design/architecture/README.md)

    * [元数据](architecture_and_design/metadata/README.md)

        * [总览](architecture_and_design/metadata/overview/README.md)

        * [元数据引擎](architecture_and_design/metadata/metadata_engine/README.md)

        * [元数据存储](architecture_and_design/metadata/metadata_storage/README.md)

    * [数据模型](architecture_and_design/data_model/README.md)

    * [SQL 处理流水线](architecture_and_design/sql_processing_pipeline/README.md)

        * [总览](architecture_and_design/sql_processing_pipeline/overview/README.md)

        * [解析器](architecture_and_design/sql_processing_pipeline/parser/README.md)

        * [绑定器](architecture_and_design/sql_processing_pipeline/binder/README.md)

        * [逻辑计划器](architecture_and_design/sql_processing_pipeline/logical_planner/README.md)

        * [优化器](architecture_and_design/sql_processing_pipeline/optimizer/README.md)

        * [物理计划器](architecture_and_design/sql_processing_pipeline/physical_planner/README.md)

        * [执行器](architecture_and_design/sql_processing_pipeline/executor/README.md)

    * [表达式求值](architecture_and_design/expression_evaluation/README.md)

        * [总览](architecture_and_design/expression_evaluation/overview/README.md)

        * [求值器与执行流程](architecture_and_design/expression_evaluation/evaluator/README.md)

        * [求值语义](architecture_and_design/expression_evaluation/semantics/README.md)

    * [函数系统](architecture_and_design/function_system/README.md)

        * [总览](architecture_and_design/function_system/overview/README.md)

        * [函数模型、注册与绑定](architecture_and_design/function_system/registry_and_binding/README.md)

        * [内置向量函数](architecture_and_design/function_system/vector_functions/README.md)

    * [存储引擎](architecture_and_design/storage_engine/README.md)

    * [标量索引](architecture_and_design/scalar_index/README.md)

    * [向量索引](architecture_and_design/vector_index/README.md)

    * [事务与恢复](architecture_and_design/transaction_and_recovery/README.md)

    * [文件系统与 IO](architecture_and_design/file_system_and_io/README.md)

        * [总览](architecture_and_design/file_system_and_io/overview/README.md)

        * [文件系统](architecture_and_design/file_system_and_io/filesystem/README.md)

        * [IO](architecture_and_design/file_system_and_io/io/README.md)

    * [网络协议](architecture_and_design/network_protocol/README.md)

        * [总览](architecture_and_design/network_protocol/overview/README.md)

        * [传输协议](architecture_and_design/network_protocol/transport_protocol/README.md)

        * [消息协议](architecture_and_design/network_protocol/message_protocol/README.md)
