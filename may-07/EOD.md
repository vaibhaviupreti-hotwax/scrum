What I did today:
- Studied business requirement for Pre Orders and explored blogs for the same (OMS redesign module)
- Revised new order sync architecture. asked ques from bot [https://shopify.dev/docs/api/admin-graphql/latest/queries/order?language=graphql] and discovered a bug: deprecated field: "LineItem.FulfillmentService".
- Discussed Product Entities with Purshottam sir and about our upcoming tasks related to product sync  

What will I do tomorrow:
- Verify and complete the Product entity understanding in the data model.
- Follow up on the execution of order lifecycle diagrams and system flows.
- Validate UserLogin entity behavior through code and database mapping.

Blockers: none

References / links:
- https://www.hotwax.co/blog/4-key-benefits-of-pre-orders-for-retailers
- https://claude.ai/chat/37ab6203-b7ba-48a2-b84d-22740604f7d1
- https://github.com/saastechacademy/foundation/blob/main/project-ideas/fulfillment-center-mgmt/readme.md
- https://chatgpt.com/c/69fae201-7e18-8321-ace7-d85e0937ed05
- https://www.arcserve.com/blog/5-common-encryption-algorithms-and-unbreakables-future
- https://github.com/saastechacademy/foundation/blob/main/udm/readme.md#chapter-2-people-and-organizations
- https://dev-oms.hotwax.io/webtools/control/FindGeneric?entityName=OrderItemShipGroup [ASSIGNMENT HELP]

Notes:
- Traced entities related to UserLogin.
- Learned CTE.
- Understood the Pre Orders business requirement (OMS redesign module) and collected resources to write the MySQL query.
- Downloaded Postman for further API testing.
- Completed Apache OFBiz 24.9 local setup and started Assignment 4.
- Read about microservices communication mechanisms and how services interact.
- Read about the meaning behind setup and build commands.
- Learned new keywords: Wave picking and SSO.
- Reviewed order lifecycle diagrams.
- Started understanding the Product entity.
- Tracked UserLogin entities from UI to the actual database.
- Visited the Apache website to check bug mirroring and issue tracking.
- Reviewed HotWax Commerce Metrics Framework materials.
- Checked fulfillment documentation and related reference links.

- The May 06 task list above is included and marked as complete in this report for tracking.
- Actual execution and verification of some items are still pending and will continue tomorrow.

- We need to understand the different scenarios and possible ways to optimize this query. [priority]
- https://dev-oms.hotwax.io/webtools/control/FindGeneric?entityName=ProductKeyword

=========================================================================================

RDS (Relational Database Service) notes:
- oms product data model vs shopify data model - mapping where to where?
- product sync
- oms tables for product sync and shopify data - origin?
- shopify object will act as a schema
- Moqui documentation: System Message Concept and MDM
- Shopify GraphQL, Query Hit, Bulk operations
- Shopify MCP and Bot
- OMS product data model notes - priority***
- Shopify product-specific data model notes
- product sync details
- system message concepts
- multiple Shopify shops in one product store - complexity
- Shopify bulk operations
- product
- prod-feature
- prod-assoc
- 14 - catalog
- 12 - prodPrice - core
- 11 -

Mentor task (separate):
- This is the task I got from mentor, kept separate after RDS.
- Focus on business requirement and product data model mapping for Pre Orders.
- Review Shopify sync behavior with OMS product model and system messages.

===================================================
communication medium - webhooks/rest api/graph ql/ web socket. 
sql
java
order LC, store LC, WH LC
Module 0 moqui assignment 
shopify integration
uml
data model - party and contact mech
what we import from shopify - integration
product and order import 

ofbiz flow and moqui flow

OFBiz: first req is rest req. job is already scheduled
sendShopifyRequest(send payload to this- order request). it is a general class. same for all rest end points
importShopifyOrders[hits rest and brings data], importShopifyOrderByID [two jobs/...]
uploadAndImportFile - read import shopify order service - java code
mdm gave config id - decides which service to call at runtime - already decided mapped kis config id ke against konsi service chalegi

process pending job - check in job mgr- chck all mdm/15 min - takes configID - iske correspond konsi service call karni hai?[iska khudka data model hai usse ye above cheez pata chalti hai - transform ka logic hai isme]

ofbiz documentation - tutorial... 

sync_shopify_orders - moqui
mother service - consume_shopify_orders - moqui
checks status - what has come and calls resp service
shopify- update history table - hash for every value is stored here, diff in hash means there is an update in order - we use update history table so we do not hit tx db again and again


webhook me sqs ki url rehti hai - 


system message ka flow

order sync ka flow old hai kayi clients 


===================================================
yes it is clear now, i am telling you everything from the beginning to end. our goal is to import orders from shopify to our dev oms environment, so first of all we subscribe to a webhook: "generally it is /updated as we dont want to pay for multiple and our req is to get orderId only due to reliability and scalability purpose. after webhook subscription, anytime change occur shopify service will ping oms via webhook json - and we will extract orderId from that", after this the main job starts, there is a service in backend that runs and creates a graphql payload to send back to shopify to extract order details - only needed- via http post request and for this we hit shopify's url.json that in response provides json response having order details. the internals  of this are: before service runs it needs data in proper format so variables are injected by xyz? - then format is made by ftl wrapper template, it maps the order id in the graphql(is it already done  - as i said?) and 2 more tasks the ftl performs swapping variables and cleaning the query and making it in a single line rather than nesting- removing spaces etc happens, then it is returned to facade which forwards this clean query back to shopify. 