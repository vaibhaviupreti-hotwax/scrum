# Session Notes - 3:00 PM (May 09)

*   **Order Component Directory**: Analyze the `order` component directory to understand how the system organizes order-related services, entities, and UI components.
*   **Order Types**: Explore the different classifications of orders in OFBiz, such as Sales Orders (customer-facing) and Purchase Orders (vendor-facing).
*   **SendSale CSR/Customer Roles**: Study the "Send Sale" business process where a CSR places an order for a customer.
    *   **Technical Detail**: Unlike a direct POS sale, a "Send Sale" is treated as a remote order and **will always require a shipping address**.
    *   **What to do**: Locate where the shipping address is stored for these orders (likely `OrderItemShipGroup` or `PostalAddress` linked via `OrderContactMech`).

pos mein bina customer ka order, no bill - db entry.
order -> return is more complex
order sync - event bridge and sqs ++ many things - oms order/types/kese aata hai order/ methods/ shipping address/ payment methods.
2 payment methods -> split payment
---
scenarios:
order test scenarios 
order adjustments
---
Edited session-3pm.md
Viewed session-3pm.md:1-17

A **Mixed Cart** order is one where the customer has items with different fulfillment requirements in a single checkout. Here are the core points:

1.  **Multiple Ship Groups**: This is the most critical technical point. The order must be split into multiple `OrderItemShipGroup` records—for example, one for "Ship to Home" items and another for "Store Pickup" (BOPIS) items.
2.  **Split Fulfillment**: The system processes parts of the order differently. The "Ship to Home" part might go through the **Brokering** engine to find the best warehouse, while the "Pickup" part is immediately routed to the specific store the customer chose.
3.  **Inventory Locking**: Inventory for pickup items is typically reserved at the chosen store immediately, while shipping items might have inventory reserved at a central warehouse.
4.  **Tax & Shipping Complexity**: Taxes may be calculated differently for items being picked up (using store location tax) vs. items being shipped (using customer's home address tax). 
5.  **Partial Completion**: A Mixed Cart order can be "Partially Completed." For example, the customer picks up their shoes at the store today (completing that part), but their jacket is still being shipped from the warehouse (keeping the overall order status as "Approved" or "Processing").
6.  **Mixed Types**: It also refers to carts containing both **Physical Products** and **Digital/Service Products** (like a Gift Card or a Warranty), which don't require shipping at all.
---
Edited session-3pm.md
Viewed session-3pm.md:20-32

A **Cash Sale** is an immediate "Take-with" transaction where the order is created, paid for, and marked as **Completed** all at once because the customer takes the product immediately. 

**How it’s different from a POS Category:**
*   **POS Category** is a way to **group products** (e.g., "Best Sellers" or "Drinks") so they show up on the POS screen for the cashier to find.
*   **Cash Sale** is the **Order Type/Flow**. You can pick a product from a "POS Category" and choose to process it as a "Cash Sale" (take now) or a "Send Sale" (ship to their home later). One is a menu organization; the other is a fulfillment method.
---
OrderAdjustment Entity
---
CashSale

mdm service - mother service - all scenarios covered here
---
order and returns scenarios - today's objective - done in actual when we will explore return service
---
sqs tool factory
methods -6: nikunj
writing in file: batch file is uploaded - data goes to MDM. 
and when it is uploaded?
deletion
---
now file is in MDM
---
mother service
==============
syncSgopifyOrder
file status in MDM?
pending->runner(running)->goes to SyncShOrder Service
processing time default - 2hrs- running 
but file payload (max 10)- per payload deafult time is 60 sec | processing and commit

file up - > 1st order- > SyncShopifyOrders.groovy[shopId and payload?]
service verb="sync"

why file is stored in mdm?

consume service internally called:
cases: order-line items/tx
pos data fiels to know which is the field

---
query MegaOrderQuery($firstOrders: Int!, $firstLineItems: Int!) {
  orders(first: $firstOrders, sortKey: CREATED_AT, reverse: true) {
    edges {
      node {
        id
        name
        createdAt
        displayFinancialStatus
        displayFulfillmentStatus

        customer {
          id
          displayName
          email
        }

        shippingAddress {
          name
          address1
          city
          countryCodeV2
        }

        lineItems(first: $firstLineItems) {
          edges {
            node {
              id
              name
              quantity
              discountedTotalSet {
                shopMoney {
                  amount
                  currencyCode
                }
              }
              variant {
                id
                title
                sku
              }
            }
          }
        }

        fulfillmentOrders(first: 10) {
          edges {
            node {
              id
              status
              location {
                id
                name
              }
            }
          }
        }
      }
    }
  }

  shop {
    id
    name
    currencyCode
  }
}
---
displayFulfillmentStatus - unfulfilled
status prepaid

---
{
  "orderId": "gid://shopify/Order/1234567890",
  "firstLineItems": 50,
  "firstFulfillmentOrders": 20,
  "firstFulfillmentOrderLineItems": 50,
  "firstReturns": 20,
  "firstReturnLineItems": 50,
  "firstExchangeLineItems": 50
}
----
currently single db
entities to bridge - 
1. check history - order available or created : if(syncHistory)
createShopifyOrder().parameters(order: createPayload)
createShopifyOrder 

return case
lineItems - 
---
{
  "data": {
    "order": {
      "id": "gid://shopify/Order/123",
      "name": "#1001",
      "currentTotalPriceSet": { "shopMoney": { "amount": "50.00", "currencyCode": "USD" } },
      "currentSubtotalLineItemsQuantity": 1,
      "lineItems": {
        "edges": [
          {
            "node": {
              "id": "gid://shopify/LineItem/111",
              "name": "T-Shirt - Blue / M",
              "quantity": 2,
              "currentQuantity": 1,
              "refundableQuantity": 0,
              "discountedTotalSet": { "shopMoney": { "amount": "40.00", "currencyCode": "USD" } }
            }
          }
        ]
      },
      "returns": {
        "edges": [
          {
            "node": {
              "id": "gid://shopify/Return/222",
              "name": "R1001",
              "status": "CLOSED",
              "totalQuantity": 1,
              "returnLineItems": {
                "edges": [
                  {
                    "node": {
                      "id": "gid://shopify/ReturnLineItem/333",
                      "quantity": 1,
                      "processedQuantity": 1,
                      "refundableQuantity": 0,
                      "withCodeDiscountedTotalPriceSet": {
                        "shopMoney": { "amount": "20.00", "currencyCode": "USD" }
                      },
                      "fulfillmentLineItem": {
                        "id": "gid://shopify/FulfillmentLineItem/444",
                        "quantity": 2,
                        "lineItem": {
                          "id": "gid://shopify/LineItem/111",
                          "name": "T-Shirt - Blue / M",
                          "quantity": 2
                        }
                      }
                    }
                  }
                ]
              },
              "refunds": {
                "edges": [
                  {
                    "node": {
                      "id": "gid://shopify/Refund/555",
                      "createdAt": "2024-01-02T12:00:00Z",
                      "totalRefundedSet": {
                        "shopMoney": { "amount": "20.00", "currencyCode": "USD" }
                      }
                    }
                  }
                ]
              }
            }
          }
        ]
      }
    }
  }
}
--- 
fulfillmentLineItems in return case: if 2 then 2 will be there

update case - what is updated?
shipping add | billing add | email | phone 
syncShopifyOrder  - line 144

cancel shipping - unfulfilled order - refund - shopify does not say this cancel | drafted as removed
refunds- cancellation flow.
order item decrease to 0 and update order as cancelled but refund not done - which is this  case?
appeasement partially refunded - $2 + refund notification - appeasement is also a return
check return data model
+
edit->add new item in same order-orderUpdate - scenario - payment not done no tx - webhook se data laa rahe hain . collect payment button ->broker->ship->delivered - money not collected(there is btn to collect money in shopify)
OPP order pay pref entity - manual Payment pref(shopify PK banata hai) and parent Payment ref
Order
--- 
OrderPaymentReference
sourceName: sh..DraftOrder
update has customer details too b/c it can contain cust updates too
customAttributes:diff specs - like christmas gift - tags - entity?

tx - update ke baad bhi sync karenge - case?
exchangeLineItem - payload is added - history is maintained - check then process
kind:refund
above case where order was placed and delivered - payment updated later(updation is always open)
---
code:

---

NEW ORDER
SYNC HISTORY CHECKED| LAUNCH DATE_G LIVE DATE - HOSTORY ON OMS - GOAL: 
WHEN HISTORY IS CREATED? MIGRATION TIME CASE?
Product Sync - 
no history - orderCreation 
OrderHeader 
shopifyRefundHistory - refundHistory.find(...)


refund - return/EXCHANGES
order creation tx / refund tx  - to link with that opp
    - refund id
    only mapping will be done later

    order split - no separate table
    ---
    this all is: syncShopifyOrder.groovy Service

---
shopifyOrderLineItemHistory

updateShopifyOrder
top down service
create| update| refund| exchange - order - to maintain orig state of sh..
when a  line item is removed ? exchanges case.

external id se item chk - error - item qty mismatch - and return not created
return sirf fulfilled items ka hoga

tx and updates
log lagakar check - payload kese aate hain

item removed - refund is created for that item
exchange for $10 (same price for both - service sequence: return / exchange order(like a new order))

there are multiple scenarios(new are created regularly) 
return exchanges cancellations









































    





