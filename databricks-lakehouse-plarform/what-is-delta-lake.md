---
cover: >-
  https://images.unsplash.com/photo-1594175268654-e60bea6c037c?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxMHx8TGFrZXxlbnwwfHx8fDE2ODU2NDgwNDl8MA&ixlib=rb-4.0.3&q=85
coverY: 0
---

# What is delta lake?

In the previous chapter, we understand the [Data Lakehouse](why-and-what-is-data-lakehouse.md#data-lakehouse) architecture and how data can be served directly from the Data Lake with comparable performance to a traditional Data Warehouse. To bring reliability, data management capabilities, and ACID transactions to data, Delta Lake was introduced. Delta Lake is an open-source storage framework/layer which enables building data lakehouse architecture. Databricks originally developed the Delta Lake protocol and continues to actively contribute to the open-source project.

In short, accessing files like you are accessing tables is what Delta Lake does.

## Why choose Delta Lake?

Running an ingestion pipeline on Cloud Storage can pose significant challenges for data teams. They often encounter issues such as difficulties in appending data accurately, making modifications to existing data, job failures halfway through, ensuring real-time operations, managing the cost of storing historical data versions, handling large metadata, dealing with the "too many files" problem, achieving optimal performance, and addressing data quality concerns. These challenges impact team efficiency and productivity, diverting focus away from business implementation.

Delta Lake addresses these technical challenges by providing a comprehensive solution that saves petabytes of data in your data lakehouse. With Delta Lake, you can simplify your data pipeline implementation while enjoying blazing-fast query responses for your BI and analytics reports.

<img src="../.gitbook/assets/file.excalidraw (10).svg" alt="" class="gitbook-drawing">

## What is Transaction Log?

Transactional Log or also known as Delta Log has the below responsibilities

* Ordering records of every transactional performed on the Table
* Acting as a single source of truth, every time we query the table Spark checks this transaction log to retrieve the most latest version of data.
* It is a collection of JSON files with commit information. It contains the operation that has been performed, for example, it's an insert or update, and the predicates such as conditions & filters used during this operation. In addition to all the files (added/removed) that have been affected because of this operation.

## Understanding Scenarios

### Write & Read

<img src="../.gitbook/assets/file.excalidraw (7).svg" alt="Write and Read" class="gitbook-drawing">

In the above scenario, there are two distinct processes: the writer process and the reader process. When the writer process initiates, it writes the Delta Lake table data into two data files, which are stored in the Parquet format. Once the writing process is completed, it appends the transaction log with the name 000.json to the \_delta\_log directory. On the other hand, when the reader process begins, it first accesses and reads the transaction log. In this particular case, it reads the transaction log named 000.json, which contains vital information about files numbered 1 and 2. Consequently, the reader process can proceed to read the corresponding files based on the details provided in the transaction log.

### Update

<img src="../.gitbook/assets/file.excalidraw (4).svg" alt="Update" class="gitbook-drawing">

In the second scenario, we encounter a writer process aiming to update a specific record residing in file number 1. However, in the context of Delta Lake, rather than modifying the existing file directly, the writer process creates a copy of file number 1 and applies the necessary updates to the new file, which we'll refer to as file number 3. Subsequently, the writer process updates the transaction log by generating a new JSON file.&#x20;

This updated transaction log indicates that file number 1 is no longer relevant for the current version of the table. Consequently, when the reader process comes into play, it accesses the transaction log, which informs it that only files 2 and 3 are associated with the current version of the table. Hence, the reader process proceeds to read and process the contents of these two files.

### Simultaneous Writes & Reads

<img src="../.gitbook/assets/file.excalidraw (9).svg" alt="Simultaneous Writes and Reads" class="gitbook-drawing">

Here, both processes want to work at the same time. The writer presses start writing file number 4. On the other hand, the reader process reads the transaction log that only has information about files 2 and 3 and not file number 4 as it is not fully written yet. So it starts reading those two files, 2 and 3 which represent the most recent data at the moment. So as we can see here, Delta Lake guarantees that we will always get the most recent version of the data. Our read operation will never have a deadlock state or conflicts with any ongoing operation on the table. Finally, the writer process finishes and it adds a new file to the log.

### Failed Writes

<img src="../.gitbook/assets/file.excalidraw (6).svg" alt="Failed Writes" class="gitbook-drawing">

Here is our last scenario. The writer presses start writing file number 5 to the lake, but this time there is an error in the job, which leads to adding the incomplete file. Because of this failure, the Delta Lake module does not write any information to the log. Now, the reader process reads the transaction log that has no information about incomplete file number 5. That's why the reader process will read-only files 2, 3, and 4. So as we can see Delta Lake guarantees that we will never read dirty data.

