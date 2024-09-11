---
title: Integrating PAIG with Milvus
---

## Prerequisites:

### 1. Configuring Milvus Collection with Dynamic Fields:
It's crucial to confirm that your existing Milvus collection is configured with the `enable_dynamic_field` parameter set to `True` during its creation phase. 
If this parameter is not set, it's necessary to recreate the collection, ensuring the inclusion of the `enable_dynamic_field=True` parameter. 

```python hl_lines="4"
schema = CollectionSchema(
    fields=[source, text, pk, vector],
    description="example_collection",
    enable_dynamic_field=True
)
```

Detailed instructions for creating a collection with this parameter enabled can be found in milvus documentation [here](https://milvus.io/docs/enable-dynamic-field.md#Enable-dynamic-field).

### 2. Adding Authorization Metadata Fields:
!!! Note "Who Should Perform This Step?"
    If the `enable_dynamic_field` parameter has not been enabled for your collection, and you prefer not to enable it, 
    you may manually add the following authorization metadata fields to your collection schema.
```python
users = FieldSchema(name="users", dtype=DataType.ARRAY, element_type=DataType.VARCHAR, max_length=65535, max_capacity=1024)
groups = FieldSchema(name="groups", dtype=DataType.ARRAY, element_type=DataType.VARCHAR, max_length=65535, max_capacity=1024)
metadata = FieldSchema(name="metadata", dtype=DataType.JSON)
```

### 3. Inserting Data with Authorization Metadata Fields:
For authorization purposes, it's essential to include the authorization metadata fields with appropriate values for each record while inserting data for each record.
Below is an example snippet of inserting data with the authorization metadata fields:

Each field has its own significance:

- `users`: Contains a list of users permitted to access the current record.
- `groups`: Contains a list of groups granted access to the current record.
- `metadata`: Comprises key-value pairs, facilitating data filtration based on defined policies within the Portal.

```python
milvus_client.insert(collection_name="example_collection", data=[
    {"source": "demo.txt", "text": "Hello World!", "vector": [0.1, 0.2, 0.3], "users": ["sally", "ryan", "john", "bob"], "groups": ["sales", "hr", "finance"], "metadata": {"security": "confidential", "country": "US"}}
])
```
Detailed instructions for inserting data with dynamic fields can be found in milvus documentation [here](https://milvus.io/docs/enable-dynamic-field.md#Insert-dynamic-data).

## Setup
To enable data governance for Milvus in your application, include the following code snippet during the application's startup phase. 
This ensures that data governance policies are consistently enforced throughout the application's lifecycle.

The code below initializes the Privacera Shield client and configures it to work with Milvus:
```python
from paig_client import client as paig_shield_client

paig_shield_client.setup(frameworks=["milvus"])
```

!!! info "Multiple Frameworks"
    This can also be utilized in conjunction with AI application policies by incorporating it within the Langchain framework, as demonstrated below:
    ```python
    from paig_client import client as paig_shield_client

    paig_shield_client.setup(frameworks=["langchain", "milvus"])
    ```

## Applying Data Governance in Milvus Operations

To ensure that data governance policies are applied during any interaction with Milvus, wrap your Milvus operations within the context of the Privacera Shield client. 
This can be done using the `create_shield_context` method, as shown below. This ensures that all operations are performed under the governance policies associated with the specified user.

```python
with privacera_shield_client.create_shield_context(username=user):
    # Your Milvus operation here
```