# Query IM Tx Status



## Get Tx status from CelerIM system

<mark style="color:blue;">`GET`</mark> `https://api.celerscan.com/scan/searchByTxHash`

[https://api.celerscan.com/scan/searchByTxHash?tx=0x0ce600fd60a7b5b3c1a3d02c7a73339261dae97d3e7785090cf8da92e657b674](https://api.celerscan.com/scan/searchByTxHash?tx=0x0ce600fd60a7b5b3c1a3d02c7a73339261dae97d3e7785090cf8da92e657b674)

#### Query Parameters

| Name                                 | Type   | Description                |
| ------------------------------------ | ------ | -------------------------- |
| tx<mark style="color:red;">\*</mark> | String | tx hash of the transaction |

{% tabs %}
{% tab title="200: OK " %}
```json
{
  "err": null,
  "txSearchInfo": [
    {
      "base_info": {
        "sender": "0x0Acd70f0Ad1C809Cc3B90Dad4a3BC3d1E82c4e47",
        "receiver": "0x0Acd70f0Ad1C809Cc3B90Dad4a3BC3d1E82c4e47",
        "src_chain_id": 42161,
        "src_tx_hash": "0x0ce600fd60a7b5b3c1a3d02c7a73339261dae97d3e7785090cf8da92e657b674",
        "init_time": "1661849935000",
        "last_update_time": "1661850036000"
      },
      "transfer": [
        {
          "xfer_id": "0x9161007dd7064a2f356a377698f7790efc7e93a1cb94016b92f27c0d0e648ed5",
          "dst_chain_id": 1,
          "send_amt": "711295513077604900",
          "received_amt": "691157868117182203",
          "src_tx_hash": "0x0ce600fd60a7b5b3c1a3d02c7a73339261dae97d3e7785090cf8da92e657b674",
          "dst_tx_hash": "0x469c084e624c25408c4a8d6c95f862a61ad71182d9b1112bbd6bb1bdd1e34b14",
          "src_token_addr": "0x82aF49447D8a07e3bd95BD0d56f35241523fBab1",
          "dst_token_addr": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
          "xfer_status": 3,
          "bridge_type": 1,
          "refund_amt": "0",
          "refund_tx": ""
        }
      ],
      "message": [
      ]
    }
  ]
}
```
{% endtab %}
{% endtabs %}

## Request Parameter

| Name | Type   | Description      |
| ---- | ------ | ---------------- |
| tx   | String | Transaction hash |

## Response Parameters

| Name         | Type                                                   | Description                    |
| ------------ | ------------------------------------------------------ | ------------------------------ |
| txSearchInfo | Array<[TxSearchInfo](query-im-tx-status.md#undefined)> | All related transactions' info |

### TxSearchInfo

|            |                                                   |                   |
| ---------- | ------------------------------------------------- | ----------------- |
| base\_info | [BaseInfo](query-im-tx-status.md#undefined)       | Basic information |
| transfer   | Array<[Transfer](query-im-tx-status.md#transfer)> | Transfers List    |
| message    | Array<[Message](query-im-tx-status.md#undefined)> | Messages List     |

### BaseInfo

| Name               | Type   | Description                          |
| ------------------ | ------ | ------------------------------------ |
| sender             | String | Sender's address                     |
| receiver           | String | Receiver's address                   |
| src\_chain\_id     | UInt32 | Source chain id                      |
| src\_tx\_hash      | String | Source chain transaction hash        |
| init\_time         | UInt64 | Initial timestamp                    |
| last\_update\_time | UInt64 | Lastest information update timestamp |

### Transfer

| Name             | Type                                          | Description                                      |
| ---------------- | --------------------------------------------- | ------------------------------------------------ |
| xfer\_id         | String                                        | cBridge transfer id                              |
| dst\_chain\_id   | UInt32                                        | Destination chain id                             |
| send\_amt        | String                                        | Source chain send amount                         |
| received\_amt    | String                                        | Destination chain receiving amount               |
| src\_tx\_hash    | String                                        | Source chain transaction hash                    |
| dst\_tx\_hash    | String                                        | Destination chain transaction hash               |
| src\_token\_addr | String                                        | Token address on source chain                    |
| dst\_token\_addr | String                                        | Token address on destination chain               |
| xfer\_status     | [XferStatus](query-im-tx-status.md#undefined) | Transfer status                                  |
| refund\_amt      | String                                        | Refund token amount on source chain. Refund only |
| refund\_tx       | String                                        | Refund transaction hash. Refund only             |

### XferStatus

| Value                                   | Description                                   |
| --------------------------------------- | --------------------------------------------- |
| XS\_UNKNOWN(0)                          | Status placeholder                            |
| XS\_WAITING\_FOR\_SGN\_CONFIRMATIONS(1) | Waiting for Celer SGN confirmation            |
| XS\_WAITING\_FOR\_FUND\_RELEASE(2)      | Waiting for fund release on destination chain |
| XS\_COMPLETED(3)                        | Complete                                      |
| XS\_TO\_BE\_REFUND(4)                   | Transfer to be refunded                       |
| XS\_REFUND\_TO\_BE\_CONFIRMED(5)        | Transfer refund to be confirmed               |
| XS\_REFUNDED(6)                         | Transfer refunded                             |

```
enum XferStatus { 
  XS_UNKNOWN = 0,
  XS_WAITING_FOR_SGN_CONFIRMATIONS = 1,
  XS_WAITING_FOR_FUND_RELEASE = 2,
  XS_COMPLETED = 3,
  XS_TO_BE_REFUND = 4,
  XS_REFUND_TO_BE_CONFIRMED = 5,
  XS_REFUNDED = 6,
}
```

### Message

| Name             | Type                                         | Description                |
| ---------------- | -------------------------------------------- | -------------------------- |
| msg\_id          | String                                       | Message id                 |
| dst\_chain\_id   | UInt32                                       | Destination chain id       |
| payload          | String                                       | payload                    |
| execution\_tx    | String                                       | Execution transaction hash |
| msg\_fee\_gas    | String                                       | Message fee gas            |
| msg\_fee\_volume | Float                                        | Message fee in USD value   |
| msg\_status      | [MsgStatus](query-im-tx-status.md#undefined) | Message Status             |

### MsgStatus

<table><thead><tr><th>Value</th><th>Description</th><th data-hidden></th></tr></thead><tbody><tr><td>MS_UNKNOWN(0)</td><td>Status placeholder</td><td></td></tr><tr><td>MS_WAITING_FOR_SGN_CONFIRMATIONS(1)</td><td>Waiting for Celer SGN confirmation</td><td></td></tr><tr><td>MS_WAITING_FOR_DESTINATION_EXECUTION(2)</td><td>Waiting for destination chain execution</td><td></td></tr><tr><td>MS_COMPLETED(3)</td><td>Complete</td><td></td></tr></tbody></table>
