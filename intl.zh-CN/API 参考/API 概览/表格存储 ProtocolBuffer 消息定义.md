# 表格存储 ProtocolBuffer 消息定义 {#reference12744 .reference}

下面是table\_store.proto和table\_store\_filter.proto的详细定义。

## table\_store.proto { .section}

```language-protobuf
package com.alicloud.openservices.tablestore.core.protocol;

message Error {
    required string code = 1;
    optional string message = 2;
}

enum PrimaryKeyType {
    INTEGER = 1;
    STRING = 2;
    BINARY = 3;
}

enum PrimaryKeyOption {
    AUTO_INCREMENT = 1;
}

message PrimaryKeySchema {
    required string name = 1;
    required PrimaryKeyType type = 2;
    optional PrimaryKeyOption option = 3; 
}

message TableOptions {
    optional int32 time_to_live = 1; // 可以动态更改
    optional int32 max_versions = 2; // 可以动态更改
	optional int64 deviation_cell_version_in_sec = 5; // 可以动态修改
}

message TableMeta {
    required string table_name = 1;
    repeated PrimaryKeySchema primary_key = 2;
}

enum RowExistenceExpectation {
    IGNORE = 0;
    EXPECT_EXIST = 1;
    EXPECT_NOT_EXIST = 2;
}

message Condition {
    required RowExistenceExpectation row_existence = 1;
    optional bytes column_condition      = 2;
}

message CapacityUnit {
    optional int32 read = 1;
    optional int32 write = 2;
}

message ReservedThroughputDetails {
    required CapacityUnit capacity_unit = 1; // 表当前的预留吞吐量的值。
    required int64 last_increase_time = 2; // 最后一次上调预留吞吐量的时间。
    optional int64 last_decrease_time = 3; // 最后一次下调预留吞吐量的时间。
}

message ReservedThroughput {
    required CapacityUnit capacity_unit = 1;
}

message ConsumedCapacity {
    required CapacityUnit capacity_unit = 1;
}

/* #############################################  CreateTable  ############################################# */
/**
 * table_meta用于存储表中不可更改的schema属性，可以更改的ReservedThroughput和TableOptions独立出来，作为UpdateTable的参数。
 * message CreateTableRequest {
 *         required TableMeta table_meta = 1;
 *         required ReservedThroughput reserved_throughput = 2;
 *         required TableOptions table_options = 3;
 * }
 */
message CreateTableRequest {
    required TableMeta table_meta = 1;
    required ReservedThroughput reserved_throughput = 2; 
    optional TableOptions table_options = 3;
}

message CreateTableResponse {
}

/* ######################################################################################################### */


/* #############################################  UpdateTable  ############################################# */
message UpdateTableRequest {
    required string table_name = 1;
    optional ReservedThroughput reserved_throughput = 2;
    optional TableOptions table_options = 3;
}

message UpdateTableResponse {
    required ReservedThroughputDetails reserved_throughput_details = 1;
    required TableOptions table_options = 2;
}
/* ######################################################################################################### */

/* #############################################  DescribeTable  ############################################# */
message DescribeTableRequest {
    required string table_name = 1;
}

message DescribeTableResponse {
    required TableMeta table_meta = 1;
    required ReservedThroughputDetails reserved_throughput_details = 2;
    required TableOptions table_options = 3;
    repeated bytes shard_splits = 6;
}
/* ########################################################################################################### */

/* #############################################  ListTable  ############################################# */
message ListTableRequest {
}

/**
 * 当前只返回一个简单的名称列表
 */
message ListTableResponse {
    repeated string table_names = 1;
}
/* ####################################################################################################### */

/* #############################################  DeleteTable  ############################################# */
message DeleteTableRequest {
    required string table_name = 1;
}

message DeleteTableResponse {
}

/* #############################################  UnloadTable  ############################################# */
message UnloadTableRequest {
    required string table_name = 1;
}

message UnloadTableResponse {

}
/* ########################################################################################################## */

/**
 * 时间戳的取值最小值为0，最大值为INT64.MAX
 * 1. 若要查询一个范围，则指定start_time和end_time
 * 2. 若要查询一个特定时间戳，则指定specific_time
 */
message TimeRange {
    optional int64 start_time = 1;
    optional int64 end_time = 2;
    optional int64 specific_time = 3;
}

/* #############################################  GetRow  ############################################# */

enum ReturnType {
    RT_NONE = 0;
    RT_PK = 1;
}

message ReturnContent {
    optional ReturnType return_type = 1;
} 

/**
 * 1. 支持用户指定版本时间戳范围或者特定的版本时间来读取指定版本的列
 * 2. 目前暂不支持行内的断点
 */
message GetRowRequest {
    required string table_name = 1;
    required bytes primary_key = 2; // encoded as InplaceRowChangeSet, but only has primary key
    repeated string columns_to_get = 3; // 不指定则读出所有的列
    optional TimeRange time_range = 4;
    optional int32 max_versions = 5;
    optional bytes filter = 7;
    optional string start_column = 8;
    optional string end_column = 9;
    optional bytes token = 10;
}

message GetRowResponse {
    required ConsumedCapacity consumed = 1;
    required bytes row = 2; // encoded as InplaceRowChangeSet
    optional bytes next_token = 3;
}
/* #################################################################################################### */

/* #############################################  UpdateRow  ############################################# */
message UpdateRowRequest {
    required string table_name = 1;
    required bytes row_change = 2;
    required Condition condition = 3;
    optional ReturnContent return_content = 4; 
}

message UpdateRowResponse {
    required ConsumedCapacity consumed = 1;
    optional bytes row = 2;
}
/* ####################################################################################################### */

/* #############################################  PutRow  ############################################# */
/**
 * 这里允许用户为每列单独设置timestamp，而不是强制整行统一一个timestamp。
 */
message PutRowRequest {
    required string table_name = 1;
    required bytes row = 2; // encoded as InplaceRowChangeSet
    required Condition condition = 3;
    optional ReturnContent return_content = 4; 
}

message PutRowResponse {
    required ConsumedCapacity consumed = 1;
    optional bytes row = 2;
}
/* #################################################################################################### */

/* #############################################  DeleteRow  ############################################# */
/**
 * OTS只支持删除指定行的所有列的所有版本。
 */
message DeleteRowRequest {
    required string table_name = 1;
    required bytes primary_key = 2; // encoded as InplaceRowChangeSet, but only has primary key
    required Condition condition = 3;
    optional ReturnContent return_content = 4; 
}

message DeleteRowResponse {
    required ConsumedCapacity consumed = 1;
    optional bytes row = 2;
}
/* ####################################################################################################### */

/* #############################################  BatchGetRow  ############################################# */
message TableInBatchGetRowRequest {
    required string table_name = 1;
    repeated bytes primary_key = 2; // encoded as InplaceRowChangeSet, but only has primary key
    repeated bytes token = 3;
    repeated string columns_to_get = 4;  // 不指定则读出所有的列
    optional TimeRange time_range = 5;
    optional int32 max_versions = 6;
    optional bytes filter = 8;
    optional string start_column = 9;
    optional string end_column = 10;
}

message BatchGetRowRequest {
    repeated TableInBatchGetRowRequest tables = 1;
}

message RowInBatchGetRowResponse {
    required bool is_ok = 1;
    optional Error error = 2;
    optional ConsumedCapacity consumed = 3;
    optional bytes row = 4; // encoded as InplaceRowChangeSet
    optional bytes next_token = 5;
}

message TableInBatchGetRowResponse {
    required string table_name = 1;
    repeated RowInBatchGetRowResponse rows = 2;
}

message BatchGetRowResponse {
    repeated TableInBatchGetRowResponse tables = 1;
}
/* ######################################################################################################### */

/* #############################################  BatchWriteRow  ############################################# */

enum OperationType {
    PUT = 1;
    UPDATE = 2;
    DELETE = 3;
}

message RowInBatchWriteRowRequest {
    required OperationType type = 1;
    required bytes row_change = 2; // encoded as InplaceRowChangeSet
    required Condition condition = 3;
    optional ReturnContent return_content = 4; 
}

message TableInBatchWriteRowRequest {
    required string table_name = 1;
    repeated RowInBatchWriteRowRequest rows = 2;
}

message BatchWriteRowRequest {
    repeated TableInBatchWriteRowRequest tables = 1;
}

message RowInBatchWriteRowResponse {
    required bool is_ok = 1;
    optional Error error = 2;
    optional ConsumedCapacity consumed = 3;
    optional bytes row = 4; 
}

message TableInBatchWriteRowResponse {
    required string table_name = 1;
    repeated RowInBatchWriteRowResponse rows = 2;
}

message BatchWriteRowResponse {
    repeated TableInBatchWriteRowResponse tables = 1;
}
/* ########################################################################################################### */

/* #############################################  GetRange  ############################################# */
enum Direction {
    FORWARD = 0;
    BACKWARD = 1;
}

message GetRangeRequest {
    required string table_name = 1;
    required Direction direction = 2;
    repeated string columns_to_get = 3;  // 不指定则读出所有的列
    optional TimeRange time_range = 4;
    optional int32 max_versions = 5;
    optional int32 limit = 6;
    required bytes inclusive_start_primary_key = 7; // encoded as InplaceRowChangeSet, but only has primary key
    required bytes exclusive_end_primary_key = 8; // encoded as InplaceRowChangeSet, but only has primary key
    optional bytes filter = 10;
    optional string start_column = 11;
    optional string end_column = 12;
    optional bytes token = 13;
}

message GetRangeResponse {
    required ConsumedCapacity consumed = 1;
    required bytes rows = 2; // encoded as InplaceRowChangeSet
    optional bytes next_start_primary_key = 3; // 若为空，则代表数据全部读取完毕. encoded as InplaceRowChangeSet, but only has primary key
    optional bytes next_token = 4;
}

/* ####################  ComputeSplitPointsBySize  #################### */
message ComputeSplitPointsBySizeRequest {
    required string table_name = 1;
    required int64 split_size = 2; // in 100MB
}

message ComputeSplitPointsBySizeResponse {
    required ConsumedCapacity consumed = 1;
    repeated PrimaryKeySchema schema = 2;

    /**
     * Split points between splits, in the increasing order
     *
     * A split is a consecutive range of primary keys,
     * whose data size is about split_size specified in the request.
     * The size could be hard to be precise.
     *
     * A split point is an array of primary-key column w.r.t. table schema,
     * which is never longer than that of table schema.
     * Tailing -inf will be omitted to reduce transmission payloads.
     */
    repeated bytes split_points = 3;

    /**
     * Locations where splits lies in.
     *
     * By the managed nature of TableStore, these locations are no more than hints.
     * If a location is not suitable to be seen, an empty string will be placed.
     */
     message SplitLocation {
         required string location = 1;
         required sint64 repeat = 2;
     }
     repeated SplitLocation locations = 4;
}

```

## table\_store\_filter.proto { .section}

```language-protobuf
package com.alicloud.openservices.tablestore.core.protocol;

enum FilterType {
    FT_SINGLE_COLUMN_VALUE = 1;
    FT_COMPOSITE_COLUMN_VALUE = 2;
    FT_COLUMN_PAGINATION = 3;
}

enum ComparatorType {
    CT_EQUAL = 1;
    CT_NOT_EQUAL = 2;
    CT_GREATER_THAN = 3;
    CT_GREATER_EQUAL = 4;
    CT_LESS_THAN = 5;
    CT_LESS_EQUAL = 6;
}

message SingleColumnValueFilter {
    required ComparatorType comparator = 1;
    required string column_name = 2;
    required bytes column_value = 3;
    required bool filter_if_missing = 4;
    required bool latest_version_only = 5; 
}

enum LogicalOperator {
    LO_NOT = 1;
    LO_AND = 2;
    LO_OR = 3;
}

message CompositeColumnValueFilter {
    required LogicalOperator combinator = 1;
    repeated Filter sub_filters = 2;
}

message ColumnPaginationFilter {
    required int32 offset = 1;
    required int32 limit = 2;
}

message Filter {
    required FilterType type = 1;
    required bytes filter = 2;  // Serialized string of filter of the type
}

```

