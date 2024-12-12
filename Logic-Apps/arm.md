<img width="929" alt="image" src="https://github.com/user-attachments/assets/e877eac8-84a2-49be-a1c8-d9c158ed1ab7" />

<img width="615" alt="image" src="https://github.com/user-attachments/assets/b80fd1f4-0b43-440e-abf7-061d3ab30ba9" />

```json
{
  "type": "object",
  "properties": {
    "orderName": { "type": "string" },
    "orderPrice": { "type": "number" }
  },
  "required": ["orderName", "orderPrice"]
}
```

```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "contentVersion": "1.0.0.0",
        "triggers": {
            "When_a_HTTP_request_is_received": {
                "type": "Request",
                "kind": "Http",
                "inputs": {
                    "method": "POST",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "orderName": {
                                "type": "string"
                            },
                            "orderPrice": {
                                "type": "number"
                            }
                        },
                        "required": [
                            "orderName",
                            "orderPrice"
                        ]
                    }
                }
            }
        },
        "actions": {
            "parseBody": {
                "runAfter": {},
                "type": "ParseJson",
                "inputs": {
                    "content": "@triggerBody()",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "orderName": {
                                "type": "string"
                            },
                            "orderPrice": {
                                "type": "number"
                            }
                        },
                        "required": [
                            "orderName",
                            "orderPrice"
                        ]
                    }
                }
            },
            "orderPriceShouldGreaterThan0": {
                "actions": {
                    "orderCreatedSuccess": {
                        "type": "Response",
                        "kind": "Http",
                        "inputs": {
                            "statusCode": 200,
                            "body": "Order Created"
                        }
                    }
                },
                "runAfter": {
                    "parseBody": [
                        "Succeeded"
                    ]
                },
                "else": {
                    "actions": {
                        "invalidOrder": {
                            "type": "Response",
                            "kind": "Http",
                            "inputs": {
                                "statusCode": 400,
                                "body": "Invalid Order"
                            }
                        }
                    }
                },
                "expression": {
                    "and": [
                        {
                            "greater": [
                                "@body('parseBody')['orderPrice']",
                                0
                            ]
                        }
                    ]
                },
                "type": "If"
            }
        },
        "outputs": {},
        "parameters": {
            "$connections": {
                "type": "Object",
                "defaultValue": {}
            }
        }
    },
    "parameters": {
        "$connections": {
            "value": {}
        }
    }
}
```
