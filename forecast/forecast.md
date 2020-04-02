## Forecast

1. Create a dataset name

2. Select a Forecast Domain

- For this example we will use revenue, sales and cashflow

3. Define the Dataset Details

```
{
    "Attributes": [
      {
        "AttributeName": "metric_name",
        "AttributeType": "string"
      },
      {
        "AttributeName": "timestamp",
        "AttributeType": "timestamp"
      },
      {
        "AttributeName": "metric_value",
        "AttributeType": "float"
      }
    ]
}
```