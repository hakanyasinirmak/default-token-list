# Varlık Token Monitoring with n8n

This guide explains how to build a simple [n8n](https://n8n.io) workflow that monitors a token named **Varlık** using the default token list that ships with this repository.

## Prerequisites

- An n8n instance (self-hosted or cloud)
- Access to the public URL that exposes the compiled token list, e.g. `https://gateway.ipfs.io/ipfs/Qm.../uni-default.tokenlist.json`
- (Optional) A Slack webhook URL if you want to receive alerts

## Workflow Overview

The workflow will:

1. Run every hour via a Cron node.
2. Download the token list JSON with an HTTP Request node.
3. Use an IF node to locate the **Varlık** token (symbol `VARLIK`).
4. Send the token metadata to a notification channel if the token exists.

## Step-by-Step

1. **Cron Node**
   - Add a Cron node to trigger the workflow every hour (or your preferred cadence).

2. **HTTP Request Node**
   - Method: `GET`
   - URL: token list endpoint (for local development you can run `yarn generate` and host `build/uniswap-default.tokenlist.json`).
   - Response Format: `JSON`

3. **Set Node (Optional)**
   - If you keep the token list locally, use a Set node to construct the URL with environment variables.

4. **Function Node**
   - Add a Function node after the HTTP Request node with the following snippet to locate the token:

     ```javascript
     const tokens = items[0].json.tokens ?? [];
     const match = tokens.find((token) => token.symbol === 'VARLIK' || token.name === 'Varlık');

     return match
       ? [{ json: match }]
       : [];
     ```

5. **IF Node**
   - Condition: `{{$json["symbol"]}}` `is not empty`
   - This ensures downstream nodes run only when the token exists in the list.

6. **Slack (or Email) Node**
   - Configure a Slack node with your webhook URL.
   - Message template example:

     ```text
     Varlık token update
     Symbol: {{$json.symbol}}
     Address: {{$json.address}}
     Chain ID: {{$json.chainId}}
     ```

7. **(Optional) Database Node**
   - Store the metadata in a database of your choice for auditing.

## Testing

- Manually execute the workflow in n8n to ensure it finds the Varlık token.
- Temporarily modify the Function node to throw an error when the token is missing to verify alerting.

## Next Steps

- Add a Price Fetch node that queries a price oracle or on-chain data.
- Enrich the notification with decimals, bridge information, or logo URI as needed.
- Combine with an HTTP Request node that fetches current liquidity metrics from Uniswap subgraphs.

