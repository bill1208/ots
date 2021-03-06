# Test model {#concept_64991_zh .concept}

-   Table structure

    |Primary key name|Type|Encoding method|Length|
    |:---------------|:---|:--------------|:-----|
    |userid|string|4-Byte-Hash + Long.toHexString|20|

-   Attribute columns

    |Attribute column name|Type|Length|
    |:--------------------|:---|:-----|
    |field0|string|100|
    |field1|string|100|
    |field2|string|100|
    |field3|string|100|
    |field4|string|100|

-   Partition quantity

    The automatic load balancing feature of Table Store dynamically splits table partitions based on the data volumes and access requests in each partition. This process requires no human intervention. In this test, we select performance data of tables typically with 1, 4, and 16 partitions.

    By default, a new data table has a single data partition. To split a new table manually, open a ticket.

-   Test cases

    Start N threads in each runner, create `com.alicloud.openservices.tablestore.SyncClient` for each thread, and call Table Store APIs.

    -   The test cases include:

        -   Random write: The test calls `SyncClient.putRow`. Each request contains one line of data and is sustained for 1 hour.
        -   Batch write: The test calls `SyncClient.batchWriteRow`. Each request contains 200 lines of data and is sustained for 1 hour.
        -   Random read: The test calls `BatchWriteRow` to write 20 GB of data to each partition, and then calls `SyncClient.getRow`. Each request gets one line of data and is sustained for 30 minutes.
        -   Random get range: The test calls `BatchWriteRow` to write 20 GB of data to each partition, and then calls `SyncClient.getRange`. Each request gets 100 lines of data and is sustained for 30 minutes.
        All test cases send requests directly to the private network address of the Table Store instance so that any effect caused by the network environment is avoided.

        This performance test is not a limit test of the service performance. The test does not trigger throttling measures on the Table Store server. The automatic load balancing feature of Table Store guarantees horizontal scale-up of the service capabilities provided by a single table. A large-scale performance test may trigger backend throttling and result in high fees. If you plan to perform a large-scale performance test, [open a ticket](https://workorder-intl.console.aliyun.com/#/ticket/createIndex) to get the test result at a reasonable cost.

        BatchWriteRow operations of Table Store are processed concurrently by partitions. The data written to each partition is a single write disk operation. We recommend that you aggregate BatchWriteRow requests by data partition keys to reduce write disk operations of each BatchWriteRow and effectively improve write performance.

        No data is written during the random read and random get range test cases. Their cache hit rates increase as the test proceeds. In low-stress scenarios, the cache hit rate increases slowly, so the test is closely related to the disk I/O capability. In high-stress scenarios, the cache hit rate increases quickly, so the test is less closely related to the disk I/O capability.

        The BatchWriteRow and GetRange test cases occupy a large amount of network bandwidth. If the performance in reading or writing your Table Store instance is lower than expected, check whether your network bandwidth has been fully occupied.

        The read performance of Table Store is significantly affected by data volumes and the cache hit rate. Some scenarios may be beyond the limit of GetRow and GetRange test cases. You can replicate the performance data generated by these two test cases. That is, you can use the data in this report as a reference in similar scenarios. If the actual read throughput, write throughput, or latency differs greatly from the data in this report, contact us to help analyze the causes.


