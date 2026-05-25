# links
- https://shopify.dev/docs/apps/build/orders-fulfillment/returns-apps/build-return-management
- https://shopify.dev/changelog/introducing-standardized-and-custom-product-types-to-graphql-admin-api
## full data reference
taxlines in shopify json
You’re likely referring to the tax_lines array that appears in Shopify’s Order JSON (REST Admin API). See the official docs here:

Order REST resource (latest) – tax_lines description and example: Order – Admin REST API
Below is a quick breakdown and examples.

What tax_lines is in Shopify order JSON
In the REST Admin API, tax_lines is an array of tax line objects that describe the taxes applied, either:

At the order level → order.tax_lines
At the line item level → order.line_items[i].tax_lines
From the docs (Order – REST):

tax_lines: An array of tax line objects, each of which details a tax applicable to the order.

Each tax line object has (at minimum) these fields:

price – Amount of tax in the shop currency, after discounts
rate – Tax rate as a decimal (e.g. 0.06 for 6%)
title – Name of the tax (e.g. "State tax")
channel_liable – Whether the sales channel is liable for remittance (true, false or null for unknown)
Example: order.tax_lines in JSON
Example from the REST docs for creating an order with order-level tax lines:
Example on order resource page

json
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
{
  "order": {
    "line_items": [
      {
        "title": "Big Brown Bear Boots",
        "price": 74.99,
        "grams": "1300",
        "quantity": 3
      }
    ],
    "transactions": [
      {
        "kind": "sale",
        "status": "success",
        "amount": 238.47
      }
    ],
    "tax_lines": [
      {
        "price": 13.5,
        "rate": 0.06,
        "title": "State tax",
        "channel_liable": null
      }
    ],
    "total_tax": 13.5,
    "currency": "EUR"
  }
}
Here:

tax_lines[0].price = 13.5 (total tax amount)
tax_lines[0].rate = 0.06 (6% tax)
tax_lines[0].title = "State tax"
Example: line_items[i].tax_lines in JSON
You can also define tax lines on individual line items:

json
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
{
  "order": {
    "line_items": [
      {
        "title": "Big Brown Bear Boots",
        "price": 74.99,
        "grams": "1300",
        "quantity": 3,
        "tax_lines": [
          {
            "price": 13.5,
            "rate": 0.06,
            "title": "State tax",
            "channel_liable": null
          }
        ]
      }
    ],
    "transactions": [
      {
        "kind": "sale",
        "status": "success",
        "amount": 238.47
      }
    ],
    "total_tax": 13.5,
    "currency": "EUR"
  }
}
Important rule from the docs:

When creating an order through the API, tax lines can be specified on the order or the line items but not both. Tax lines specified on the order are split across the taxable line items in the created order.

Source: Order REST resource – tax_lines description

How this maps to the Admin GraphQL API
If you’re working with GraphQL instead of raw JSON:

REST order.tax_lines roughly maps to GraphQL Order.taxLines
REST line_items[i].tax_lines maps to OrderLineItem.taxLines
You can query them with something like:

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
query OrderTaxLines($id: ID!) {
  order(id: $id) {
    name
    totalTaxSet {
      shopMoney {
        amount
        currencyCode
      }
    }
    taxLines {
      rate
      priceSet {
        shopMoney {
          amount
          currencyCode
        }
      }
      title
      channelLiable
    }
  }
}
If you’d like, tell me which of these you’re trying to do:

Read tax lines from existing orders (REST or GraphQL)?
Create orders with custom tax lines?
Map/transform tax_lines into another system?
And I can give you a more specific JSON or GraphQL example tailored to that.

Sources
API: Admin REST > Latest > Resources > Order
Order – Admin REST API
Was this answer useful?

Yes

No
customProduct and its handling
There are two different “custom” concepts in Shopify that often get confused:

Custom product type on the Product object (GraphQL: customProductType)
Custom line items / custom products on an order or draft order (REST: line item with "custom": true)
From your wording (“customProduct and its handling”), you most likely mean the GraphQL Admin API’s customProductType field, introduced here:

Introducing standardized and custom product types to GraphQL Admin API
General product model overview: About products and collections
I’ll explain that first. If instead you meant “custom line items on draft orders / orders”, let me know and I’ll cover that in detail too.

1. What is customProductType in Shopify Admin GraphQL?
The GraphQL Admin API Product object has two different product type fields:

standardizedProductType
customProductType
This is described in the changelog:
Introducing standardized and custom product types to GraphQL Admin API

Standardized type:

Links a product to the Shopify Product Taxonomy (a controlled, standardized set of categories).
Used by Shopify for features such as:
Eligibility for Shop app
Reporting and analytics features (e.g., BFCM Notebook).
You don’t invent this string – you pick from Shopify’s product taxonomy.
Custom type:

A free-form string that you define yourself.
Gives you more granular or business‑specific categorization than the standardized taxonomy.
Can be used for your app’s internal logic, filters, reporting, etc.
Both fields can co‑exist:

standardizedProductType = “Tops > T-Shirts”
customProductType = “Limited Edition Collab 2025”
2. How to read customProductType via GraphQL (Admin API)
You can fetch the custom product type along with core product fields.

The main product docs: Admin GraphQL API – Product object

Example query (get product title + custom product type):

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
query GetProductCustomProductType($id: ID!) {
  product(id: $id) {
    id
    title
    customProductType
    standardizedProductType {
      productTaxonomyNode {
        fullName
      }
    }
  }
}
Typical variables:

json
Copy
1
2
3
{
  "id": "gid://shopify/Product/1234567890"
}
Explanation:

customProductType: your custom label for this product.
standardizedProductType.productTaxonomyNode.fullName: the full name from the Shopify taxonomy (e.g., “Clothing > Tops > T-Shirts”).
3. How to set or update customProductType via GraphQL
You’ll generally use productUpdate (or productCreate when creating a product) and pass customProductType in the input.

3.1. Update an existing product’s custom type
Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
mutation SetCustomProductType($input: ProductInput!) {
  productUpdate(input: $input) {
    product {
      id
      title
      customProductType
    }
    userErrors {
      field
      message
    }
  }
}
Variables example:

json
Copy
1
2
3
4
5
6
{
  "input": {
    "id": "gid://shopify/Product/1234567890",
    "customProductType": "Premium Hoodie – Winter 2026"
  }
}
Behavior:

If a customProductType was already set, this overwrites it.
If it was null, it becomes "Premium Hoodie – Winter 2026".
3.2. Remove / clear the custom product type
You can set it to null in the input to clear it:

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
mutation ClearCustomProductType($input: ProductInput!) {
  productUpdate(input: $input) {
    product {
      id
      title
      customProductType
    }
    userErrors {
      field
      message
    }
  }
}
Variables:

json
Copy
1
2
3
4
5
6
{
  "input": {
    "id": "gid://shopify/Product/1234567890",
    "customProductType": null
  }
}
4. How does customProductType relate to the old productType string?
Historically, Shopify products just had a single productType string. Over time, that split logically into:

A standardized taxonomy-linked type (standardizedProductType)
A custom free-text type (customProductType)
In Admin GraphQL:

productType still exists for backward compatibility.
Shopify’s direction is to move toward:
standardizedProductType + customProductType
rather than overloading a single field.
When designing a new app:

Use standardizedProductType when you want consistency across stores and to leverage Shopify features.
Use customProductType for store‑ or app‑specific labels and logic.
5. “Custom products” vs customProductType
To avoid confusion:

customProductType (GraphQL Product field)

A string field on the Product object.
Used for categorization / tagging / taxonomy.
“Custom product/line item” (REST draft orders & orders)

In REST, a line item on a draft order can be “custom” and not linked to a real Product/Variant.
Appears as:
json
Copy
1
2
3
4
5
6
     {
       "title": "Custom Tee",
       "price": "20.00",
       "quantity": 1,
       "custom": true
     }
Example snippet from the draft order REST docs:
DraftOrder REST resource
If you were actually asking about how to create and handle those custom line items (not tied to a real product) in GraphQL or REST, tell me:

Are you using REST or Admin GraphQL?
Is the context draft orders, orders, or cart/checkout?
Then I can give you concrete examples like:

REST: POST /admin/api/2026-04/draft_orders.json with "line_items": [{"custom": true, ...}]
GraphQL: using draft order mutations and the appropriate input types.
6. Next step – what kind of handling do you need?
Depending on your use case, we can:

Bulk set customProductType for many products (GraphQL bulk operation).
Keep customProductType in sync with an external catalog.
Use customProductType in queries/filters (e.g., search products by custom type).
Or handle custom line items in draft orders (different concept).
Reply with:

Whether you meant customProductType field on products or
custom line items / custom products on orders/draft orders, and
What you want to do (read, write, sync, filter, etc.),
and I’ll give you targeted GraphQL operations or JSON payload examples.

Sources
Changelog > Introducing Standardized And Custom Product Types To Graphql Admin API
Introducing standardized and custom product types to GraphQL Admin API
Apps > Build > Product Merchandising > Products And Collections
About products and collections
API: Admin GraphQL > Latest > Objects > Product
Admin GraphQL API – Product object
show 2 more
Was this answer useful?

Yes

No
restock? 4..5 lines
In Shopify, “restock” usually refers to adding inventory back when items are returned or an order is canceled.
In the REST Admin API, this is controlled via properties like restock_type on a refund / order adjustment (e.g. no_restock, cancel, legacy restock).
When you create a refund or cancel a fulfillment, you specify whether to restock the items so inventory is increased at the associated location.
See: Order REST API – refunds & restocking behavior.

Sources
API: Admin REST > Latest > Resources > Order
Order REST API – refunds & restocking behavior
Was this answer useful?

Yes

No
we get 2 objects from shopify return and refund but dont create return from return, answer in 2 lines
Shopify sends both a Return (movement of items back to inventory) and a Refund (financial transaction), but they’re separate system objects describing different aspects of the same event.
You should treat the Return as authoritative for item movement and the Refund as authoritative for money, and never create a new Return from an existing Return—just link to the original Return/Refund IDs.

Was this answer useful?

Yes

No
 and never create a new Return from an existing Return— //try again , 2lines
Shopify already creates the Return object that represents item movement, and the Refund object that represents the money movement.
Your system should consume those records as-is and must not generate an additional Return based on an incoming Return, to avoid double-counting stock.

Was this answer useful?

Yes

No
any other scenario for this decision? 2 liner
If you cloned Returns, you’d also risk triggering duplicate workflows (RMA processing, customer notifications, or warehouse tasks) based on the same physical return.
Additionally, any later updates from Shopify (edited return quantities, cancellations) would be impossible to reconcile cleanly if you’d created your own shadow Return objects.

Was this answer useful?

Yes

No
  "pricing": {
      "subtotalPrice": "245.00",
      "totalDiscounts": "0.00",
      "totalTax": "0.00",
      "shippingPrice": "0.00",
      "totalPrice": "245.00",
      "totalOutstanding": "0.00",
      "currency": "USD"
    },
 //what if there are 4 line items and 1 return 3 of them, will shipment charges be counted in return? or is this charge non refundable? 2..3 lines
Shipping is not automatically prorated into the return; by default, only the returned line items’ value (and their tax) are refunded.
Whether shipping is refunded is a separate decision: you can choose to refund shipping fully/partially (or not at all) when creating the Refund/Return in Shopify.

Was this answer useful?

Yes

No
Whether shipping is refunded is a separate decision: //tell me this scenario//2lines//4lines
Example: Customer bought 4 items + $10 shipping, returns 3 items.
You approve a refund for the 3 items only, keep the $10 shipping non‑refundable (customer still pays shipping).

Different policy: same order and return, but you decide to be generous.
You refund the value of the 3 items plus the full $10 shipping (or part of it), explicitly including shipping in the refund.

Was this answer useful?

Yes

No
so basically it is non refund..? yes no
By default, yes – shipping is non‑refundable unless you explicitly choose to refund it.
So: not automatically refunded, but you can manually/refund it (fully or partially) based on your policy.

Was this answer useful?

Yes

No
when is it refunded
It’s refunded only when the merchant (or app) explicitly includes shipping in the refund/return—e.g. in the Shopify Admin “Refund” screen, checking “Refund shipping” or entering a shipping amount.
Some flows (like canceling an unfulfilled order and choosing to refund in full) will also include shipping because the refund is for the entire order, not just specific items.

Was this answer useful?

Yes

No
return shopify graphql/json?
For returns, Shopify’s Admin GraphQL API uses a Return object in queries/mutations, and responses are always standard JSON.
Example from the returns guide: the returnReopen mutation returns JSON like:

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
mutation returnReopenMutation {
  returnReopen(id: "gid://shopify/Return/1") {
    return { id }
    userErrors { field message }
  }
}
json
Copy
1
2
3
4
5
6
7
8
{
  "data": {
    "returnReopen": {
      "return": { "id": "gid://shopify/Return/1" },
      "userErrors": []
    }
  }
}
So:

GraphQL = how you request or modify returns (Return type, returnCreate, returnReopen, etc.).
JSON = the format Shopify uses to send back the result (including for return webhooks).
Docs: Build for return management (returns, refunds, reverse fulfillment, webhooks)

Sources
Apps > Build > Orders Fulfillment > Returns Apps > Build Return Management
Build for return management (returns, refunds, reverse fulfillment, webhooks)
Was this answer useful?

Yes

No
apptitle?
If you mean the Shopify app’s title/name (what merchants see in the Admin and in the App Store):

It’s set in your app configuration in the Partners Dashboard and exposed via the app’s metadata, not via a Shopify Admin GraphQL field like apptitle.
In embedded apps, you usually hard‑code or configure the app title in your frontend (e.g., your Polaris <AppProvider> / navigation) rather than fetching it from Shopify.
If you meant appTitle or app-related fields in some specific JSON / GraphQL response (e.g., webhooks, metafields, returns), paste that snippet and I can explain exactly what that field is.

Was this answer useful?

Yes

No
yes in retun
In the Return context, appTitle (or app_title in some JSON payloads) is simply the name of the app that created or is managing that return.

If a return is created by a returns app, appTitle will be that app’s name (e.g. “Acme Returns Portal”), so you can attribute the action to that app.
If the return is created directly in the Shopify Admin (no third‑party app), you’ll typically see a core identifier like “Shopify” instead.
This is mainly for auditing, attribution, and reporting—it doesn’t affect pricing, refunds, or stock calculations.

You can see how returns and apps tie together in the returns guide:
Build for return management (GraphQL Admin API + webhooks).

Sources
Apps > Build > Orders Fulfillment > Returns Apps > Build Return Management
Build for return management (GraphQL Admin API + webhooks)
