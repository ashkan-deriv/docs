# Balance API - AsyncAPI Documentation

## Overview

This directory contains the AsyncAPI 3.0.0 schema documentation for the Balance API, which provides WebSocket endpoints for retrieving account balances and subscribing to real-time balance updates.

## Files

- **balance_request.schema.json** - JSON Schema for balance request messages
- **balance_response.schema.json** - JSON Schema for balance response messages
- **../../asyncapi.json** - AsyncAPI 3.0.0 document for the Balance API (root level)

## API Description

The Balance API allows clients to:

1. **Retrieve Current Balance** - Get the current account balance with currency and login information
2. **Subscribe to Updates** - Receive real-time notifications whenever the balance changes due to transactions, trades, deposits, or withdrawals

### WebSocket Endpoint

- **Demo Server**: `wss://demov2.derivws.com/websockets/v3`
- **Protocol**: WebSocket Secure (WSS)
- **Content Type**: application/json

**Note**: This is a single WebSocket endpoint. All API operations (balance, authorize, trades, etc.) are sent as JSON messages over the same WebSocket connection, not as separate URL endpoints.

## Request Format

### Get Balance (One-time)

```json
{
  "balance": 1,
  "req_id": 1
}
```

### Get Balance with Subscription

```json
{
  "balance": 1,
  "subscribe": 1,
  "req_id": 2
}
```

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `balance` | integer | Yes | Must be `1` to request balance |
| `subscribe` | integer | No | Set to `1` to receive real-time updates |
| `req_id` | integer | No | Optional request ID for mapping responses |
| `passthrough` | object | No | Optional data to pass through |

## Response Format

### Balance Response (One-time)

```json
{
  "balance": {
    "balance": 10000.50,
    "currency": "USD",
    "loginid": "CR123456"
  },
  "echo_req": {
    "balance": 1,
    "req_id": 1
  },
  "msg_type": "balance",
  "req_id": 1
}
```

### Balance Response (With Subscription)

```json
{
  "balance": {
    "balance": 10000.50,
    "currency": "USD",
    "id": "sub_123abc",
    "loginid": "CR123456"
  },
  "subscription": {
    "id": "sub_123abc"
  },
  "echo_req": {
    "balance": 1,
    "subscribe": 1,
    "req_id": 2
  },
  "msg_type": "balance",
  "req_id": 2
}
```

### Balance Update (Real-time)

When subscribed, you'll receive updates like this whenever the balance changes:

```json
{
  "balance": {
    "balance": 9950.25,
    "currency": "USD",
    "id": "sub_123abc",
    "loginid": "CR123456"
  },
  "subscription": {
    "id": "sub_123abc"
  },
  "echo_req": {
    "balance": 1,
    "subscribe": 1
  },
  "msg_type": "balance"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `balance.balance` | number | Current account balance |
| `balance.currency` | string | Currency code (USD, EUR, BTC, etc.) |
| `balance.loginid` | string | Client login ID |
| `balance.id` | string | Subscription ID (only when subscribed) |
| `subscription.id` | string | Unique subscription identifier |
| `echo_req` | object | Echo of the request made |
| `msg_type` | string | Always "balance" for this API |
| `req_id` | integer | Request ID if provided in request |

## Mintlify Integration

The Balance API is integrated with Mintlify's documentation and includes an interactive WebSocket playground.

### Configuration

The API is configured in `docs.json`:

```json
{
  "navigation": {
    "tabs": [
      {
        "tab": "API reference",
        "asyncapi": "/asyncapi.json",
        "groups": [
          {
            "group": "WebSocket API",
            "pages": []
          }
        ]
      }
    ]
  },
  "api": {
    "playground": {
      "mode": "show"
    }
  }
}
```

### Using the Playground

1. **Navigate** to the API Reference → WebSocket API section in your documentation
2. **Connect** to the WebSocket server (wss://ws.binaryws.com/websockets/v3)
3. **Send** a balance request message
4. **View** the real-time response
5. **Subscribe** to receive live balance updates

## Validation

You can validate the AsyncAPI schema using:

- **AsyncAPI Studio**: https://studio.asyncapi.com/
  - Paste the contents of `asyncapi.json` to validate
- **AsyncAPI CLI**: `asyncapi validate asyncapi.json`

## Security

The Balance API requires authentication via a session token. The token should be obtained through:

1. OAuth flow
2. API token generation from the dashboard

Include the token in an `authorize` message after establishing the WebSocket connection.

## Example Usage

### JavaScript/TypeScript

```javascript
const ws = new WebSocket('wss://demov2.derivws.com/websockets/v3');

ws.onopen = () => {
  // First authorize (if required)
  ws.send(JSON.stringify({
    authorize: 'your_session_token'
  }));
  
  // Then request balance with subscription
  ws.send(JSON.stringify({
    balance: 1,
    subscribe: 1,
    req_id: 1
  }));
};

ws.onmessage = (event) => {
  const response = JSON.parse(event.data);
  
  if (response.msg_type === 'balance') {
    console.log('Balance:', response.balance.balance);
    console.log('Currency:', response.balance.currency);
  }
};
```

### Python

```python
import websocket
import json

def on_message(ws, message):
    data = json.loads(message)
    if data.get('msg_type') == 'balance':
        print(f"Balance: {data['balance']['balance']}")
        print(f"Currency: {data['balance']['currency']}")

def on_open(ws):
    # Request balance with subscription
    ws.send(json.dumps({
        'balance': 1,
        'subscribe': 1,
        'req_id': 1
    }))

ws = websocket.WebSocketApp(
    'wss://demov2.derivws.com/websockets/v3',
    on_message=on_message,
    on_open=on_open
)

ws.run_forever()
```

## Unsubscribing

To stop receiving balance updates, use the `forget` API with the subscription ID:

```json
{
  "forget": "sub_123abc"
}
```

## Error Handling

If an error occurs, the response will include an `error` object:

```json
{
  "error": {
    "code": "InvalidToken",
    "message": "Invalid session token"
  },
  "echo_req": {
    "balance": 1
  },
  "msg_type": "balance"
}
```

## References

- [AsyncAPI 3.0.0 Specification](https://www.asyncapi.com/docs/reference/specification/v3.0.0)
- [Mintlify AsyncAPI Setup](https://www.mintlify.com/docs/api-playground/asyncapi/setup)
- [Mintlify AsyncAPI Playground](https://www.mintlify.com/docs/api-playground/asyncapi/playground)

## Support

For API support, contact:
- Email: api@deriv.com
- URL: https://api.deriv.com

