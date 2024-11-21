---
title: Troubleshoot the Azure Cosmos DB connector
description: Learn how to troubleshoot issues with the Azure Cosmos DB connector in Data Factory in Microsoft Fabric.
ms.reviewer: jburchel
ms.author: xupzhou
author: pennyzhou-msft
ms.topic: troubleshooting
ms.custom:
  - build-2023
  - ignite-2023
  - ignite-2024
ms.date: 10/23/2024
---

# Troubleshoot the Azure Cosmos DB connector in Data Factory in Microsoft Fabric

This article provides suggestions to troubleshoot common problems with the Azure Cosmos DB connector in Data Factory in Microsoft Fabric.

## Error message: Request size is too large

- **Message**: `Request size is too large.`

- **Cause**: Azure Cosmos DB limits the size of a single request to 2 MB. The formula is request size = single document size * write batch size. If your document size is large, the default behavior will result in a request size that's too large.

- **Recommendation**: You can tune the write batch size. In the copy activity destination, reduce the *write batch size* value (the default value is 10000).

  If reducing the *write batch size* value to 1 still doesn't work, change your Cosmos DB authentication type to use service principal or system-assigned managed identity or user-assigned managed identity. This authentication enables the connection to use Azure Cosmos DB SQL API V3, which supports larger request sizes.

## Error message: Unique index constraint violation

- **Symptoms**: When you copy data into Azure Cosmos DB, you receive the following error:

    `Message=Partition range id 0 | Failed to import mini-batch. 
    Exception was Message: {"Errors":["Encountered exception while executing function. Exception = Error: {\"Errors\":[\"Unique index constraint violation.\"]}...`

- **Cause**: There are two possible causes:

    - Cause 1: If you use **Insert** as the write behavior, this error means that your source data has rows or objects with same ID.
    - Cause 2: If you use **Upsert** as the write behavior and you set another unique key to the container, this error means that your source data has rows or objects with different IDs but the same value for the defined unique key.

- **Resolution**: 

    - For cause 1, set **Upsert** as the write behavior.
    - For cause 2, make sure that each document has a different value for the defined unique key.

## Error message: Request rate is large

- **Symptoms**: When you copy data into Azure Cosmos DB, you receive the following error:

    `Type=Microsoft.Azure.Documents.DocumentClientException,
    Message=Message: {"Errors":["Request rate is large"]}`

- **Cause**: The number of used request units (RUs) is greater than the available RUs configured in Azure Cosmos DB. To learn how
Azure Cosmos DB calculates RUs, see [Request units in Azure Cosmos DB](/azure/cosmos-db/request-units#request-unit-considerations).

- **Resolution**: Try either of the following two solutions:

    - Increase the *container RUs* number to a greater value in Azure Cosmos DB. This solution will improve the copy activity performance, but it will incur more cost in Azure Cosmos DB. 
    - Decrease *writeBatchSize* to a lesser value, such as 1000, and decrease *parallelCopies* to a lesser value, such as 1. This solution will reduce copy run performance, but it won't incur more cost in Azure Cosmos DB.

## Columns missing in column mapping

- **Symptoms**: When you import a schema for Azure Cosmos DB for column mapping, some columns are missing. 

- **Cause**: Azure Data Factory and Synapse pipelines infer the schema from the first 10 Azure Cosmos DB documents. If some document columns or properties don't contain values, the schema isn't detected and consequently isn't displayed.

- **Resolution**: You can tune the query as shown in the following code to force the column values to be displayed in the result set with empty values. Assume that the *impossible* column is missing in the first 10 documents). Alternatively, you can manually add the column for mapping.

    ```sql
    select c.company, c.category, c.comments, (c.impossible??'') as impossible from c
    ```

## Error message: The GuidRepresentation for the reader is CSharpLegacy

- **Symptoms**: When you copy data from Azure Cosmos DB MongoAPI or MongoDB with the universally unique identifier (UUID) field, you receive the following error:

    `Failed to read data via MongoDB client., 
    Source=Microsoft.DataTransfer.Runtime.MongoDbV2Connector,Type=System.FormatException, 
    Message=The GuidRepresentation for the reader is CSharpLegacy which requires the binary sub type to be UuidLegacy not UuidStandard.,Source=MongoDB.Bson,’“,`

- **Cause**: There are two ways to represent the UUID in Binary JSON (BSON): UuidStardard and UuidLegacy. By default, UuidLegacy is used to read data. You will receive an error if your UUID data in MongoDB is UuidStandard.

- **Resolution**: In the MongoDB connection string, add the *uuidRepresentation=standard* option. For more information, see [MongoDB connection string](/azure/data-factory/connector-mongodb#linked-service-properties).

## Error code: CosmosDbSqlApiOperationFailed

- **Message**: `CosmosDbSqlApi operation Failed. ErrorMessage: %msg;.`

- **Cause**: A problem with the CosmosDbSqlApi operation.  This applies to the Azure Cosmos DB for NoSQL connector specifically.

- **Recommendation**:  To check the error details, see [Azure Cosmos DB help document](/azure/cosmos-db/troubleshoot-dot-net-sdk). For further help, contact the Azure Cosmos DB team.

## Error code: CosmosDbSqlApiPartitionKeyExceedStorage

- **Message**: `The size of data each logical partition can store is limited, current partitioning design and workload failed to store more than the allowed amount of data for a given partition key value.`

- **Cause**: The data size of each logical partition is limited, and the partition key reached the maximum size of your logical partition.

- **Recommendation**: Check your Azure Cosmos DB partition design. For more information, see [Logical partitions](/azure/cosmos-db/partitioning-overview#logical-partitions).

## Related content

For more troubleshooting help, try these resources:

- [Data Factory blog](https://blog.fabric.microsoft.com/blog/category/data-factory)
- [Data Factory community](https://community.fabric.microsoft.com/t5/Data-Factory-preview-Community/ct-p/datafactory)
- [Data Factory feature requests ideas](https://ideas.fabric.microsoft.com/)