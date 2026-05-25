# Tathya

SELECT * FROM System_Message_Type
WHERE SYSTEM_MESSAGE_TYPE_ID LIKE '%bulk%'
;

SELECT * FROM System_Message_Type
WHERE description not LIKE '%system%'
;

SELECT DISTINCT STATUS_ID FROM System_Message;

---
# group
maarg/runtime/component/shopify-oms-bridge/data/SOBSystemMessageTypeData.xml       272 line number

---

 <!-- SystemMessageType record for Shopify Order Sync -->
    <moqui.service.message.SystemMessageType systemMessageTypeId="ShopifyOrderSync"
            description="Shopify Order Sync"
            sendServiceName="co.hotwax.sob.order.OrderFeedServices.send#ShopifyOrderSync"/>
---

 <moqui.service.message.SystemMessageType systemMessageTypeId="ShopifyOrderSync"
            description="Shopify Order Sync"
            sendServiceName="co.hotwax.sob.order.OrderFeedServices.send#ShopifyOrderSync"/>
</entity-facade-xml>

---

    <!-- SystemMessageType record for Shopify Order Sync -->
    <moqui.service.message.SystemMessageType systemMessageTypeId="ShopifyOrderSync"
         description="Shopify Order Sync"
         sendServiceName="co.hotwax.sob.order.OrderFeedServices.send#ShopifyOrderSync"/>

---
{
"sendServiceName": "co.hotwax.sob.order.OrderFeedServices.send#ShopifyOrderSync",
"lastUpdatedStamp": "2026-04-21T10:47:37+0000",
"systemMessageTypeId": "ShopifyOrderSync",
"createdStamp": "2026-02-20T05:05:41+0000",
"description": "Shopify Order Sync",
"_entity": "moqui.service.message.SystemMessageType"
}

---
{
"sendServiceName": "co.hotwax.shopify.system.ShopifySystemMessageServices.send#ShopifyBulkQueryMessage",
"produceServiceName": "co.hotwax.shopify.product.ShopifyProductServices.sync#ShopifyProductUpdates",
"lastUpdatedStamp": "2026-05-05T04:19:52+0000",
"systemMessageTypeId": "BulkQueryShopifyProductUpdates",
"parentTypeId": "ShopifyBulkQuery",
"consumeServiceName": "co.hotwax.shopify.system.ShopifySystemMessageServices.consume#ShopifyProductDataFile",
"receiveMovePath": "runtime://datamanager/shopify/BulkQueryShopifyProductUpdates",
"createdStamp": "2026-04-24T06:25:00+0000",
"description": "Product and Variants updates query for Shopify",
"_entity": "moqui.service.message.SystemMessageType"
}

https://dev-maarg.hotwax.io/qapps/tools/Entity/DataEdit/EntityDataEdit?systemMessageTypeId=BulkQueryShopifyProductUpdates&pageIndex=0&selectedEntity=moqui.service.message.SystemMessageType

---

 <moqui.service.message.SystemMessageType systemMessageTypeId="BulkOrderCustomAttributesQuery"
            description="Bulk Order Custom Attributes Query System Message"
            parentTypeId="ShopifyBulkQuery"
            sendServiceName="co.hotwax.shopify.system.ShopifySystemMessageServices.send#BulkQuerySystemMessage"
            sendPath="component://shopify-connector/template/graphQL/BulkOrderCustomAttributesQuery.ftl"
            consumeServiceName="co.hotwax.shopify.system.ShopifySystemMessageServices.consume#BulkOperationResult"
            receivePath="${contentRoot}/shopify/BulkOrderCustomAttributesQuery/BulkOperationResult-${systemMessageId}-${remoteMessageId}-${nowDate}.jsonl">
        <parameters parameterName="consumeSmrId" parameterValue="" systemMessageRemoteId=""/>
    </moqui.service.message.SystemMessageType>

---

<!-- Parent SystemMessageType for all the shopify bulk query system message types -->
    <moqui.service.message.SystemMessageType systemMessageTypeId="ShopifyBulkQuery" description="Parent SystemMessageType for Shopify Bulk Query"/>

/home/adarsh/src/hotwax/training/sandbox/maarg/runtime/component/mantle-shopify-connector/data/ShopifySetupSeedData.xml
---

Prerak sir 

https://docs.google.com/spreadsheets/d/10OohRmWsZg-Gb--Z13unCqCb5F-mf_rnfxz9cDfj_X8/edit?gid=1639984740#gid=1639984740




--

my screen
https://dev-maarg.hotwax.io/qapps/tools/Entity/DataEdit/EntityDataEdit?systemMessageTypeId=BulkQueryShopifyProductUpdates&pageIndex=0&selectedEntity=moqui.service.message.SystemMessageType

https://nextgen-maarg.hotwax.io/qapps/tools/Entity/DataEdit/EntityList?filterRegexp=systemmess&viewOption=all

https://dev-maarg.hotwax.io/qapps/tools/Entity/DataEdit/EntityDataEdit?systemMessageTypeId=BulkQueryShopifyProductUpdates&pageIndex=0&selectedEntity=moqui.service.message.SystemMessageType


