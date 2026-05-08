## Everything from the beginning to end. 
End Goal: To import orders from shopify to our dev oms environment.
First of all we subscribe to a webhook: "generally it is /updated as we dont want to pay for multiple and our req is to get orderId only due to reliability and scalability purpose. after webhook subscription, anytime change occur shopify service will ping oms via webhook json - and we will extract orderId from that", after this the main job starts, there is a service in backend that runs and creates a graphql payload to send back to shopify to extract order details - only needed- via http post request and for this we hit shopify's url.json that in response provides json response having order details. the internals of this are: before service runs it needs data in proper format so variables are injected by xyz? - then format is made by ftl wrapper template, it maps the order id in the graphql(is it already done - as i said?) and 2 more tasks the ftl performs swapping variables and cleaning the query and making it in a single line rather than nesting- removing spaces etc happens, then it is returned to facade which forwards this clean query back to shopify.

---

# Technical Overview: Shopify to OMS Order Synchronization

This document outlines the end-to-end process of importing orders from Shopify into the OMS environment, emphasizing reliability, scalability, and precise data retrieval.

## 1. Webhook Notification & Order Identification

The synchronization process is triggered by Shopify webhooks, treating them as lightweight "pings" rather than data payloads.

*   **Webhook Subscription**: The system subscribes to the `orders/updated` topic.
*   **Rationale**:
    *   **Reliability**: Webhooks can be delayed or arrive out of order. By only using the `orderId`, we ensure the subsequent data fetch retrieves the most current state directly from Shopify.
    *   **Scalability**: Minimizes the overhead of processing large REST payloads for every update.
*   **Action**: Upon receiving the webhook, the OMS verifies the authenticity and extracts the `admin_graphql_api_id` (e.g., `gid://shopify/Order/1234567890`).

## 2. Internal Processing & Query Construction

Once the order ID is identified, a background service prepares the request to fetch detailed order information.

### Data Preparation
A background service (e.g., `createOrdersFromShopify`) identifies the orders pending sync and prepares the execution context. This involves injecting necessary variables like the `orderId` into the process.

### FTL Template (GraphQL Payload Builder)
A Freemarker (`.ftl`) template acts as a wrapper to construct the final GraphQL query. It performs the following critical tasks:
1.  **Variable Injection**: Maps the dynamic `orderId` into the GraphQL query structure.
2.  **Query Cleaning & Compression**: Uses `<@compress single_line=true>` to remove unnecessary whitespace and line breaks.
3.  **Final Output**: Produces a clean, single-line GraphQL string optimized for transmission over HTTP.

## 3. Transmission (The Facade Layer)

The "Facade" component acts as the interface between the internal OMS logic and the Shopify API.

*   **Role**: It receives the cleaned GraphQL query from the FTL layer, wraps it in a JSON body (including variables if applicable), and executes an **HTTP POST** request.
*   **Endpoint**: `https://{shop}.myshopify.com/admin/api/{version}/graphql.json`
*   **Authentication**: Securely includes the `X-Shopify-Access-Token` in the request headers.

---

# Shopify GraphQL Reference: `order` Query

The `order` query is the primary mechanism for retrieving detailed information about a specific order using its global GraphQL ID.

### Query Purpose
Returns an `Order` object for the specified ID. This object provides access to comprehensive data, including line items, fulfillment status, financial details, and customer information.

### Arguments
| Argument | Type | Description |
| :--- | :--- | :--- |
| `id` | `ID!` | The unique global ID of the order (e.g., `gid://shopify/Order/1234567890`). |

### Key Fields for Synchronization
When importing orders, the following fields are typically fetched:
*   `name`: The order number (e.g., `#1001`).
*   `createdAt`: Timestamp of order creation.
*   `totalPriceSet`: Total price in shop and presentment currencies.
*   `displayFinancialStatus`: The current financial state of the order.
*   `displayFulfillmentStatus`: The current fulfillment state.
*   `lineItems`: List of products, quantities, and pricing in the order.
*   `shippingAddress`: The destination for the order.

### Example GraphQL Implementation

```graphql
query GetOrderDetails($id: ID!) {
  order(id: $id) {
    id
    name
    createdAt
    displayFinancialStatus
    displayFulfillmentStatus
    totalPriceSet {
      shopMoney {
        amount
        currencyCode
      }
    }
    lineItems(first: 50) {
      edges {
        node {
          title
          quantity
          sku
        }
      }
    }
  }
}
```

### JSON Response Structure
The response is parsed by the OMS to map Shopify data into the internal database schema.

```json
{
  "data": {
    "order": {
      "id": "gid://shopify/Order/1234567890",
      "name": "#1024",
      "createdAt": "2023-10-27T12:00:00Z",
      "displayFinancialStatus": "PAID",
      "displayFulfillmentStatus": "UNFULFILLED",
      "lineItems": {
        "edges": [
          {
            "node": {
              "title": "Example Product",
              "quantity": 1,
              "sku": "EX-PROD-01"
            }
          }
        ]
      }
    }
  }
}
```