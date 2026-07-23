# Task: Deprecate service-backed sales order detail read
## Bopis app
- [/src/store/order.ts] getOrderDetail(): uses oms/orders/${orderId}
```
async getOrderDetail({ payload, orderType }: { payload: any, orderType: string }) {
      const orderId = payload.orderId;
      let currentOrder = {} as any;

      const productStore = useProduct();

      try {
        const resp = await api({
          url: `oms/orders/${orderId}`,
          method: "get",
        });
//other code lines
  ```
- **testing the new api** response from Postman application
- checking **base url** http://localhost:8080/rest/s1/ to test   
- created new branch, took pull from main. branch: **585-deprecate-service-backed-url**
- faced error: login needed
```json
{
    "errorCode": 403,
    "errors": "User [No User] is not authorized for View on REST Path /oms/orders"
}
```
- login api
```ts
useAuth.ts:L85
):
URL: POST http://localhost:8080/rest/s1/admin/login
Method: POST (Must be POST, GET returns 404)
Headers: Content-Type: application/json
Body (raw -> JSON)
```
- Moqui Check Login Options (see localApiServerDiscovery.ts:L52):
## JSON TEST RESPONSE [REST]: http://localhost:8080/rest/s1/oms/orders?orderId=M100000&dependencyLevels=1
```JSON
[
    {
        "salesChannelEnumId": "WEB_SALES_CHANNEL",
        "notes": [
            {
                "lastUpdatedStamp": 1780032073769,
                "noteId": "M100000",
                "internalNote": "Y",
                "_entity": "org.apache.ofbiz.order.order.OrderHeaderNote"
            }
        ],
        "orderId": "M100000",
        "roles": [
            {
                "fromDate": 1780032079079,
                "lastUpdatedStamp": 1780032073769,
                "roleTypeId": "BILL_FROM_VENDOR",
                "partyGroup": {
                    "lastUpdatedStamp": 1784575836263,
                    "groupName": "Default Company",
                    "logoImageUrl": "/resources/uploads/images/company_logo.png",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "COMPANY",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1780032079079,
                "lastUpdatedStamp": 1780032073769,
                "roleTypeId": "SHIP_FROM_VENDOR",
                "partyGroup": {
                    "lastUpdatedStamp": 1784575836263,
                    "groupName": "Default Company",
                    "logoImageUrl": "/resources/uploads/images/company_logo.png",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "COMPANY",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1780032079079,
                "lastUpdatedStamp": 1780032073769,
                "roleTypeId": "BILL_TO_CUSTOMER",
                "person": {
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782707008783
                },
                "partyGroup": {
                    "lastUpdatedStamp": 1784575835467,
                    "groupName": "Default",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "_NA_",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1780032079078,
                "lastUpdatedStamp": 1780032073769,
                "roleTypeId": "END_USER_CUSTOMER",
                "person": {
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782707008783
                },
                "partyGroup": {
                    "lastUpdatedStamp": 1784575835467,
                    "groupName": "Default",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "_NA_",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1780032079071,
                "lastUpdatedStamp": 1780032073769,
                "roleTypeId": "PLACING_CUSTOMER",
                "person": {
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782707008783
                },
                "partyGroup": {
                    "lastUpdatedStamp": 1784575835467,
                    "groupName": "Default",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "_NA_",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1780032079079,
                "lastUpdatedStamp": 1780032073769,
                "roleTypeId": "SHIP_TO_CUSTOMER",
                "person": {
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782707008783
                },
                "partyGroup": {
                    "lastUpdatedStamp": 1784575835467,
                    "groupName": "Default",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "_NA_",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            }
        ],
        "orderTypeId": "SALES_ORDER",
        "paymentPreferences": [
            {
                "presentmentAmount": 885.95,
                "_entity": "org.apache.ofbiz.order.order.OrderPaymentPreference",
                "orderPaymentPreferenceId": "M100000",
                "presentmentCurrencyUom": "USD",
                "moqui.basic.StatusItem": {
                    "lastUpdatedStamp": 1784575833974,
                    "statusId": "PAYMENT_SETTLED",
                    "statusAge": 100,
                    "statusTypeId": "PAYMENT_PREF_STATUS",
                    "description": "Settled",
                    "statusCode": "SETTLED",
                    "_entity": "statuses"
                },
                "maxAmount": 885.95,
                "lastUpdatedStamp": 1780032073769,
                "org.apache.ofbiz.accounting.payment.PaymentMethodType": {
                    "description": "Ext Other Gateways",
                    "lastUpdatedStamp": 1779945143228,
                    "paymentMethodTypeId": "EXT_SHOP_OTHR_GTWAY",
                    "_entity": "org.apache.ofbiz.accounting.payment.PaymentMethodType"
                },
                "manualRefNum": "6576470917437",
                "paymentMethodTypeId": "EXT_SHOP_OTHR_GTWAY",
                "createdDate": 1780032081786,
                "statusId": "PAYMENT_SETTLED"
            }
        ],
        "autoApprove": "N",
        "_entity": "org.apache.ofbiz.order.order.OrderHeader",
        "presentmentCurrencyUom": "USD",
        "productStoreId": "STORE",
        "orderName": "#1001",
        "lastUpdatedStamp": 1780032073769,
        "entryDate": 1780032076252,
        "grandTotal": 885.95,
        "externalId": "5429950382397",
        "identifications": [
            {
                "fromDate": 1780032076265,
                "lastUpdatedStamp": 1780032073769,
                "idValue": "5429950382397",
                "orderIdentificationTypeId": "SHOPIFY_ORD_ID",
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1780032076265,
                "lastUpdatedStamp": 1780032073769,
                "idValue": "#1001",
                "orderIdentificationTypeId": "SHOPIFY_ORD_NAME",
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1780032076265,
                "lastUpdatedStamp": 1780032073769,
                "idValue": "1001",
                "orderIdentificationTypeId": "SHOPIFY_ORD_NO",
                "_entity": "co.hotwax.order.OrderIdentification"
            }
        ],
        "originFacilityId": "_NA_",
        "shipGroups": [
            {
                "lastUpdatedStamp": 1780032073769,
                "facilityId": "_NA_",
                "maySplit": "Y",
                "shipGroupSeqId": "00001",
                "_entity": "org.apache.ofbiz.order.order.OrderItemShipGroup",
                "carrierPartyId": "_NA_",
                "carrierRoleTypeId": "CARRIER",
                "isGift": "N",
                "shipmentMethodTypeId": "STANDARD",
                "items": [
                    {
                        "_entity": "org.apache.ofbiz.order.order.OrderItem",
                        "itemDescription": "Default Title",
                        "orderItemSeqId": "01",
                        "unitPrice": 885.95,
                        "productId": "M100000",
                        "taxCode": "true",
                        "requestedShipMethTypeId": "STANDARD",
                        "statusId": "ITEM_CREATED",
                        "statuses": [
                            {
                                "lastUpdatedStamp": 1780032073769,
                                "orderStatusId": "M100000",
                                "statusId": "ITEM_CREATED",
                                "statusDatetime": 1687908059000,
                                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
                            }
                        ],
                        "isPromo": "N",
                        "isModifiedPrice": "Y",
                        "unitListPrice": 885.95,
                        "orderItemTypeId": "PRODUCT_ORDER_ITEM",
                        "lastUpdatedStamp": 1780032073769,
                        "quantity": 1,
                        "externalId": "14023977664829",
                        "prodCatalogId": "CATALOG"
                    }
                ]
            }
        ],
        "statusId": "ORDER_CREATED",
        "currencyUom": "USD",
        "localeString": "en",
        "contents": [
            {
                "orderItemSeqId": "_NA_",
                "fromDate": 1780032076265,
                "lastUpdatedStamp": 1780032073769,
                "contentId": "M100000",
                "orderContentTypeId": "ORDER_STATUS_URL",
                "_entity": "org.apache.ofbiz.order.order.OrderContent"
            }
        ],
        "statuses": [
            {
                "orderItemSeqId": "01",
                "lastUpdatedStamp": 1780032073769,
                "orderStatusId": "M100000",
                "statusId": "ITEM_CREATED",
                "statusDatetime": 1687908059000,
                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
            },
            {
                "lastUpdatedStamp": 1780032073769,
                "orderStatusId": "M100001",
                "statusId": "ORDER_CREATED",
                "statusDatetime": 1687908059000,
                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
            }
        ],
        "orderDate": 1687908059000
    }
]
```
## JSON TEST RESPONSE [SERVICE-BACKED]: http://localhost:8080/rest/s1/oms/orders/M100000
```JSON
{
    "orderDetail": {
        "orderId": "M100000",
        "facilityExternalId": "_NA_",
        "facilityTypeDesc": "Not Available",
        "salesChannel": "WEB_CHANNEL",
        "statusDatetime": 1687908059000,
        "_entity": "co.hotwax.warehouse.OrderItemShipmentDetail",
        "parentFacilityTypeId": "VIRTUAL_FACILITY",
        "partyId": "_NA_",
        "orderItemSeqId": "01",
        "unitPrice": 885.95,
        "productId": "M100000",
        "entryDate": 1780032076252,
        "grandTotal": 885.95,
        "shipGroupSeqId": "00001",
        "orderItemExternalId": "14023977664829",
        "orderExternalId": "5429950382397",
        "orderStatusId": "ORDER_CREATED",
        "shipGroups": [
            {
                "shipGroupSeqId": "00001",
                "shipmentMethodTypeId": "STANDARD",
                "shipmentMethodTypeDesc": "Standard",
                "shipmentId": null,
                "shipmentStatusId": null,
                "trackingCode": null,
                "carrierPartyId": null,
                "facilityId": "_NA_",
                "facilityName": "Brokering Queue",
                "facilityTypeId": "NA",
                "facilityTypeDescription": "Not Available",
                "parentFacilityTypeId": "VIRTUAL_FACILITY",
                "picklistId": null,
                "picklistDate": null,
                "pickerId": null,
                "pickerFirstName": null,
                "pickerLastName": null,
                "pickerGroupName": null,
                "items": [
                    {
                        "orderId": "M100000",
                        "facilityExternalId": "_NA_",
                        "facilityTypeDesc": "Not Available",
                        "salesChannel": "WEB_CHANNEL",
                        "statusDatetime": 1687908059000,
                        "_entity": "co.hotwax.warehouse.OrderItemShipmentDetail",
                        "parentFacilityTypeId": "VIRTUAL_FACILITY",
                        "sku": null,
                        "orderItemSeqId": "01",
                        "unitPrice": 885.95,
                        "productId": "M100000",
                        "entryDate": 1780032076252,
                        "grandTotal": 885.95,
                        "shipGroupSeqId": "00001",
                        "orderItemExternalId": "14023977664829",
                        "orderExternalId": "5429950382397",
                        "orderStatusId": "ORDER_CREATED",
                        "returnableQuantity": 1,
                        "orderDate": 1687908059000,
                        "internalName": "45528481661245",
                        "facilityTypeId": "NA",
                        "currency": "USD",
                        "productStoreId": "STORE",
                        "facilityName": "Brokering Queue",
                        "orderName": "#1001",
                        "customerPartyId": "_NA_",
                        "roleTypeId": "BILL_TO_CUSTOMER",
                        "slaShipmentMethodTypeId": "STANDARD",
                        "isPhysical": "Y",
                        "itemQuantity": 1,
                        "facilityId": "_NA_",
                        "quantity": 1,
                        "isDigital": "N",
                        "maySplit": "Y",
                        "itemStatusId": "ITEM_CREATED",
                        "productTypeId": "FINISHED_GOOD",
                        "alreadyReturnedQuantity": 0
                    }
                ]
            }
        ],
        "currencyUom": "USD",
        "statuses": [
            {
                "orderId": "M100000",
                "orderItemSeqId": null,
                "lastUpdatedStamp": null,
                "orderStatusId": null,
                "orderPaymentPreferenceId": null,
                "statusUserLogin": null,
                "changeReason": null,
                "createdStamp": null,
                "statusId": "ORDER_CREATED",
                "changeReasonEnumId": null,
                "statusDatetime": null
            }
        ],
        "orderDate": 1687908059000,
        "orderEmail": null,
        "paymentPreferences": [
            {
                "maxAmount": 885.95,
                "cardName": null,
                "statusId": "PAYMENT_SETTLED",
                "manualAuthCode": null,
                "presentmentAmount": null,
                "orderId": "M100000",
                "lastModifiedByUserLogin": null,
                "orderPaymentPreferenceId": null,
                "exchangeRate": null,
                "paymentMethodId": null,
                "presentmentCurrencyUom": null,
                "requestId": null,
                "overflowFlag": null,
                "productPricePurposeId": null,
                "createdByUserLogin": null,
                "orderItemSeqId": null,
                "track2": null,
                "paymentMethodTypeId": "EXT_SHOP_OTHR_GTWAY",
                "statusDesc": "Settled",
                "lastModifiedDate": null,
                "paymentMode": null,
                "manualRefNum": null,
                "swipedFlag": null,
                "securityCode": null,
                "createdDate": 1780032081786,
                "parentRefNum": null,
                "shipGroupSeqId": null,
                "applicationIdentifier": null,
                "paymentMethodTypeDesc": "Ext Other Gateways",
                "processAttempt": null,
                "needsNsfRetry": null,
                "finAccountId": null,
                "billingPostalCode": null,
                "presentFlag": null
            }
        ],
        "internalName": "45528481661245",
        "billingEmail": null,
        "facilityTypeId": "NA",
        "currency": "USD",
        "productStoreId": "STORE",
        "facilityName": "Brokering Queue",
        "orderName": "#1001",
        "customerPartyId": "_NA_",
        "roleTypeId": "BILL_TO_CUSTOMER",
        "slaShipmentMethodTypeId": "STANDARD",
        "isPhysical": "Y",
        "itemQuantity": 1,
        "facilityId": "_NA_",
        "isDigital": "N",
        "maySplit": "Y",
        "shippingPhone": null,
        "itemStatusId": "ITEM_CREATED",
        "productTypeId": "FINISHED_GOOD",
        "shippingAddress": {},
        "attributes": [],
        "billingAddress": {},
        "billingPhone": null,
        "shippingEmail": null
    }
}
```
# SERVICE VS REST API COMPARISON
Here is a detailed comparison between the **service-backed endpoint** (`GET /oms/orders/{orderId}`) and the **new master REST endpoint** (`GET /oms/orders?orderId={orderId}&dependentLevels=1`), along with the exact code updates needed in [order.ts](file:///home/vaibhaviupreti/1-HW/freq-used/13-accxui/apps/bopis/src/store/order.ts#L374-L440).

---

### 🔍 Key Differences Between the Two Payloads

| Aspect | Service-Backed (`GET /oms/orders/M100000`) | New Master Route (`GET /oms/orders?orderId=M100000...`) |
| :--- | :--- | :--- |
| **Response Wrapper** | Wrapped in an object: `{ "orderDetail": { ... } }` | Wrapped in an array: `[ { ... } ]` |
| **Order External ID** | `data.orderExternalId` | `data.externalId` |
| **Order Status ID** | `data.orderStatusId` | `data.statusId` |
| **Customer ID** | `data.partyId` | Found in `data.roles` array where `roleTypeId === 'BILL_TO_CUSTOMER'` |
| **Customer Name** | `data.customerFirstName`, `data.customerLastName` | Inside `data.roles` (`person.firstName`, `person.lastName` or `partyGroup.groupName`) |
| **Item Quantity** | `item.itemQuantity` or `item.quantity` | `item.quantity` |
| **Item Status ID** | `item.itemStatusId` | `item.statusId` (e.g. `ITEM_CREATED`, `ITEM_CANCELLED`) |

---

### 🛠️ Will it affect the flow?

**Yes, if not mapped properly.** The main flow would break because:
1. `resp.data.orderDetail` is `undefined` in the new response (`resp.data` is an array `[ { ... } ]`).
2. `orderStatusId` vs `statusId` name differences.
3. `orderExternalId` vs `externalId` name differences.
4. `itemStatusId` vs `statusId` name differences.

---

### 📝 How to Update `getOrderDetail` in [order.ts](file:///home/vaibhaviupreti/1-HW/freq-used/13-accxui/apps/bopis/src/store/order.ts#L380-L415)

Update [order.ts](file:///home/vaibhaviupreti/1-HW/freq-used/13-accxui/apps/bopis/src/store/order.ts#L380-L415) with backwards-compatible fallbacks so it works seamlessly with the new REST payload:

```typescript
if (resp.status === 200 && !commonUtil.hasError(resp) && resp.data) {
  // 1. Extract order data (handles both array [0] from new route and orderDetail object from old route)
  const data = Array.isArray(resp.data) 
    ? resp.data[0] 
    : (resp.data.orders?.[0] || resp.data.orderDetail || resp.data);

  if (!data) return;

  const productIds = data.shipGroups.flatMap((group: any) =>
    group.items.map((item: any) => item.productId)
  );
  await productStore.fetchProducts({ productIds });

  const sortedPaymentPreference = [...(data.paymentPreferences || [])].sort((a: any, b: any) => (b.createdDate || 0) - (a.createdDate || 0));

  const shipGroups = data.shipGroups.map((group: any) => {
    const validItems = group.items.filter(
      (item: any) => (item.statusId || item.itemStatusId) !== 'ITEM_CANCELLED'
    );

    return {
      ...group,
      category: orderUtil.getOrderCategory(group),
      items: orderUtil.removeKitComponents(validItems).map((item: any) => ({
        ...item,
        showKitComponents: false
      }))
    };
  });

  // Extract Customer Role if needed
  const customerRole = data.roles?.find((r: any) => r.roleTypeId === 'BILL_TO_CUSTOMER' || r.roleTypeId === 'PLACING_CUSTOMER');
  const customerFirstName = data.customerFirstName || customerRole?.person?.firstName || "";
  const customerLastName = data.customerLastName || customerRole?.person?.lastName || "";
  const customerName = (`${customerFirstName} ${customerLastName}`).trim() || customerRole?.partyGroup?.groupName || "";

  const order = {
    ...data,
    shipGroups,
    statusId: data.statusId || data.orderStatusId,
    customerId: data.partyId || data.customerPartyId || customerRole?.partyId,
    customerName,
    shopifyOrderId: data.externalId || data.orderExternalId,
    approvedDate: data.statuses?.find((status: any) => status.statusId === "ORDER_APPROVED")?.statusDatetime,
    completedDate: data.statuses?.find((status: any) => status.statusId === "ORDER_COMPLETED")?.statusDatetime,
    paymentPreferences: sortedPaymentPreference
  };
```

- faced an issue during "user-login": user is not associated with any facility, checked facility_party, still not working -- blocker,
- for above set: VITE_OMS_TYPE=MOQUI | checked cache and pinia and storage too(that was okay) | user.ts(userState working fine - checked browser storage), checked oms: "" (data filled at login time), added maarg
- continued
- faced a blocker: data was not present - data setup done
- edited an order to make it bopis
- solr issue
  <img width="681" height="181" alt="image" src="https://github.com/user-attachments/assets/e22e0ce8-4a53-47a0-8850-ecb8bbb0b75a" />
- Completed Solr Configuration: http://localhost:8080/qapps/Oms/Settings/Search/SearchAdmin?selectedCollection=enterpriseSearch, updated facility
- updated code according to new api, tested and found test failed
- <img width="1212" height="429" alt="image" src="https://github.com/user-attachments/assets/3f6d3bec-8558-4a16-9203-c355598b8776" />



## updated rest json
```json
[
    {
        "salesChannelEnumId": "WEB_SALES_CHANNEL",
        "orderId": "M106429",
        "createdStamp": 1782174500169,
        "roles": [
            {
                "fromDate": 1782174504340,
                "lastUpdatedStamp": 1782174500169,
                "roleTypeId": "BILL_FROM_VENDOR",
                "createdStamp": 1782174500169,
                "partyGroup": {
                    "lastUpdatedStamp": 1779690572277,
                    "groupName": "Default Company",
                    "createdStamp": 1779735446249,
                    "logoImageUrl": "/resources/uploads/images/company_logo.png",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "COMPANY",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1782174504340,
                "lastUpdatedStamp": 1782174500169,
                "roleTypeId": "SHIP_FROM_VENDOR",
                "createdStamp": 1782174500169,
                "partyGroup": {
                    "lastUpdatedStamp": 1779690572277,
                    "groupName": "Default Company",
                    "createdStamp": 1779735446249,
                    "logoImageUrl": "/resources/uploads/images/company_logo.png",
                    "_entity": "org.apache.ofbiz.party.party.PartyGroup"
                },
                "partyId": "COMPANY",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1782174504340,
                "lastUpdatedStamp": 1782174500169,
                "roleTypeId": "BILL_TO_CUSTOMER",
                "person": {
                    "lastName": "Wilson",
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782174740212,
                    "firstName": "Wade"
                },
                "createdStamp": 1782174500169,
                "partyId": "M111424",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1782174504339,
                "lastUpdatedStamp": 1782174500169,
                "roleTypeId": "END_USER_CUSTOMER",
                "person": {
                    "lastName": "Wilson",
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782174740212,
                    "firstName": "Wade"
                },
                "createdStamp": 1782174500169,
                "partyId": "M111424",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1782174504337,
                "lastUpdatedStamp": 1782174500169,
                "roleTypeId": "PLACING_CUSTOMER",
                "person": {
                    "lastName": "Wilson",
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782174740212,
                    "firstName": "Wade"
                },
                "createdStamp": 1782174500169,
                "partyId": "M111424",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            },
            {
                "fromDate": 1782174504339,
                "lastUpdatedStamp": 1782174500169,
                "roleTypeId": "SHIP_TO_CUSTOMER",
                "person": {
                    "lastName": "Wilson",
                    "_entity": "org.apache.ofbiz.party.party.Person",
                    "lastUpdatedStamp": 1782174740212,
                    "firstName": "Wade"
                },
                "createdStamp": 1782174500169,
                "partyId": "M111424",
                "_entity": "org.apache.ofbiz.order.order.OrderRole"
            }
        ],
        "orderTypeId": "SALES_ORDER",
        "paymentPreferences": [
            {
                "createdStamp": 1782174500169,
                "presentmentAmount": 69,
                "_entity": "org.apache.ofbiz.order.order.OrderPaymentPreference",
                "orderPaymentPreferenceId": "M100613",
                "presentmentCurrencyUom": "USD",
                "moqui.basic.StatusItem": {
                    "lastUpdatedStamp": 1779690571782,
                    "statusId": "PAYMENT_AUTHORIZED",
                    "statusAge": 50,
                    "createdStamp": 1779735445523,
                    "statusTypeId": "PAYMENT_PREF_STATUS",
                    "description": "Authorized",
                    "statusCode": "AUTHORIZED",
                    "_entity": "statuses"
                },
                "maxAmount": 69,
                "lastUpdatedStamp": 1782174500169,
                "org.apache.ofbiz.accounting.payment.PaymentMethodType": {
                    "description": "Ext Other Gateways",
                    "lastUpdatedStamp": 1779690572318,
                    "paymentMethodTypeId": "EXT_SHOP_OTHR_GTWAY",
                    "createdStamp": 1779735445985,
                    "_entity": "org.apache.ofbiz.accounting.payment.PaymentMethodType"
                },
                "manualRefNum": "8761635832125",
                "paymentMethodTypeId": "EXT_SHOP_OTHR_GTWAY",
                "createdDate": 1782174505041,
                "statusId": "PAYMENT_AUTHORIZED"
            }
        ],
        "autoApprove": "Y",
        "_entity": "org.apache.ofbiz.order.order.OrderHeader",
        "presentmentCurrencyUom": "USD",
        "billToPartyId": "M111424",
        "productStoreId": "STORE",
        "orderName": "#HCD1332",
        "lastUpdatedStamp": 1783942002484,
        "entryDate": 1782174503239,
        "grandTotal": 69,
        "externalId": "72121248975979",
        "identifications": [
            {
                "fromDate": 1783945930118,
                "lastUpdatedStamp": 1783947284652,
                "createdStamp": 1783945930217,
                "idValue": "RTL_1234",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "thruDate": 1783947284635,
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1783948318631,
                "lastUpdatedStamp": 1783955121595,
                "createdStamp": 1783948318661,
                "idValue": "RET_1234567",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "thruDate": 1783955121533,
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1783955121533,
                "lastUpdatedStamp": 1783955238530,
                "createdStamp": 1783955121613,
                "idValue": "RET_12345678",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "thruDate": 1783955238450,
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1783955238450,
                "lastUpdatedStamp": 1783955304123,
                "createdStamp": 1783955238599,
                "idValue": "RET_12345678",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "thruDate": 1783955304066,
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1783955304066,
                "lastUpdatedStamp": 1783956262197,
                "createdStamp": 1783955304139,
                "idValue": "RET_123456789",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "thruDate": 1783956262111,
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1783956262111,
                "lastUpdatedStamp": 1784020487716,
                "createdStamp": 1783956262197,
                "idValue": "RET_1234567890",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "thruDate": 1784020487676,
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1784020487676,
                "lastUpdatedStamp": 1784020487751,
                "createdStamp": 1784020487751,
                "idValue": "RET_123456",
                "orderIdentificationTypeId": "RETAIL_PRO_ORDER_ID",
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1782174503298,
                "lastUpdatedStamp": 1783952311675,
                "createdStamp": 1782174500169,
                "idValue": "7212124897597",
                "orderIdentificationTypeId": "SHOPIFY_ORD_ID",
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1782174503298,
                "lastUpdatedStamp": 1782174500169,
                "createdStamp": 1782174500169,
                "idValue": "#HCD1332",
                "orderIdentificationTypeId": "SHOPIFY_ORD_NAME",
                "_entity": "co.hotwax.order.OrderIdentification"
            },
            {
                "fromDate": 1782174503298,
                "lastUpdatedStamp": 1783951391492,
                "createdStamp": 1782174500169,
                "idValue": "1332",
                "orderIdentificationTypeId": "SHOPIFY_ORD_NO",
                "_entity": "co.hotwax.order.OrderIdentification"
            }
        ],
        "originFacilityId": "_NA_",
        "contactMechs": [
            {
                "lastUpdatedStamp": 1782174500169,
                "contactMech": {
                    "lastUpdatedStamp": 1782174500169,
                    "contactMechTypeId": "EMAIL_ADDRESS",
                    "infoString": "banibrata.manna@hotwaxsystems.com",
                    "createdStamp": 1782174500169,
                    "_entity": "org.apache.ofbiz.party.contact.ContactMech"
                },
                "contactMechPurposeTypeId": "ORDER_EMAIL",
                "contactMechId": "M102457",
                "createdStamp": 1782174500169,
                "_entity": "org.apache.ofbiz.order.order.OrderContactMech"
            },
            {
                "lastUpdatedStamp": 1782174500169,
                "contactMech": {
                    "lastUpdatedStamp": 1782174500169,
                    "contactMechTypeId": "EMAIL_ADDRESS",
                    "infoString": "banibrata.manna@hotwaxsystems.com",
                    "createdStamp": 1782174500169,
                    "_entity": "org.apache.ofbiz.party.contact.ContactMech"
                },
                "contactMechPurposeTypeId": "SHIPPING_EMAIL",
                "contactMechId": "M102460",
                "createdStamp": 1782174500169,
                "_entity": "org.apache.ofbiz.order.order.OrderContactMech"
            },
            {
                "lastUpdatedStamp": 1782174500169,
                "contactMech": {
                    "lastUpdatedStamp": 1782174500169,
                    "contactMechTypeId": "POSTAL_ADDRESS",
                    "createdStamp": 1782174500169,
                    "_entity": "org.apache.ofbiz.party.contact.ContactMech"
                },
                "postalAddress": {
                    "countryGeoId": "USA",
                    "lastUpdatedStamp": 1782174500169,
                    "city": "Austin",
                    "address1": "Wade Wilson, 8104 Dunn St #AAustin, Suite 100",
                    "postalCode": "78745",
                    "latitude": 30.2006003,
                    "createdStamp": 1782174500169,
                    "_entity": "org.apache.ofbiz.party.contact.PostalAddress",
                    "toName": "Wade Wilson",
                    "stateProvinceGeoId": "TX",
                    "geoPointId": "M100615",
                    "longitude": -97.8348133
                },
                "contactMechPurposeTypeId": "SHIPPING_LOCATION",
                "createdStamp": 1782174500169,
                "contactMechId": "M102459",
                "_entity": "org.apache.ofbiz.order.order.OrderContactMech"
            }
        ],
        "shipGroups": [
            {
                "lastUpdatedStamp": 1784728523667,
                "facilityId": "_NA_",
                "createdStamp": 1782174480000,
                "maySplit": "Y",
                "shipGroupSeqId": "00001",
                "contactMechId": "M102459",
                "_entity": "org.apache.ofbiz.order.order.OrderItemShipGroup",
                "carrierPartyId": "_NA_",
                "carrierRoleTypeId": "CARRIER",
                "isGift": "N",
                "shipmentMethodTypeId": "STOREPICKUP"
            },
            {
                "lastUpdatedStamp": 1784729277925,
                "facilityId": "BROOKLYN",
                "createdStamp": 1782174480000,
                "maySplit": "Y",
                "shipGroupSeqId": "00002",
                "contactMechId": "M102459",
                "_entity": "org.apache.ofbiz.order.order.OrderItemShipGroup",
                "carrierPartyId": "_NA_",
                "carrierRoleTypeId": "CARRIER",
                "isGift": "N",
                "shipmentMethodTypeId": "STOREPICKUP",
                "items": [
                    {
                        "createdStamp": 1782174500169,
                        "_entity": "org.apache.ofbiz.order.order.OrderItem",
                        "itemDescription": "XS / Blue",
                        "orderItemSeqId": "01",
                        "unitPrice": 69,
                        "productId": "M100121",
                        "taxCode": "true",
                        "requestedShipMethTypeId": "STANDARD",
                        "statusId": "ITEM_APPROVED",
                        "statuses": [
                            {
                                "lastUpdatedStamp": 1782174500169,
                                "orderStatusId": "M101668",
                                "statusId": "ITEM_CREATED",
                                "createdStamp": 1782174500169,
                                "statusDatetime": 1782174464000,
                                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
                            },
                            {
                                "lastUpdatedStamp": 1782174500169,
                                "orderStatusId": "M101671",
                                "statusId": "ITEM_APPROVED",
                                "statusUserLogin": "system",
                                "createdStamp": 1782174500169,
                                "statusDatetime": 1782174504654,
                                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
                            }
                        ],
                        "isPromo": "N",
                        "isModifiedPrice": "Y",
                        "unitListPrice": 69,
                        "orderItemTypeId": "PRODUCT_ORDER_ITEM",
                        "lastUpdatedStamp": 1782259251145,
                        "quantity": 1,
                        "externalId": "17431636705597",
                        "prodCatalogId": "CATALOG"
                    }
                ]
            }
        ],
        "statusId": "ORDER_APPROVED",
        "currencyUom": "USD",
        "localeString": "en",
        "contents": [
            {
                "orderItemSeqId": "_NA_",
                "fromDate": 1782174503299,
                "lastUpdatedStamp": 1782174500169,
                "createdStamp": 1782174500169,
                "contentId": "M100717",
                "orderContentTypeId": "ORDER_STATUS_URL",
                "_entity": "org.apache.ofbiz.order.order.OrderContent"
            }
        ],
        "statuses": [
            {
                "orderItemSeqId": "01",
                "lastUpdatedStamp": 1782174500169,
                "orderStatusId": "M101668",
                "statusId": "ITEM_CREATED",
                "createdStamp": 1782174500169,
                "statusDatetime": 1782174464000,
                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
            },
            {
                "lastUpdatedStamp": 1782174500169,
                "orderStatusId": "M101669",
                "statusId": "ORDER_CREATED",
                "createdStamp": 1782174500169,
                "statusDatetime": 1782174464000,
                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
            },
            {
                "lastUpdatedStamp": 1782174500169,
                "orderStatusId": "M101670",
                "statusId": "ORDER_APPROVED",
                "statusUserLogin": "system",
                "createdStamp": 1782174500169,
                "statusDatetime": 1782174504650,
                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
            },
            {
                "orderItemSeqId": "01",
                "lastUpdatedStamp": 1782174500169,
                "orderStatusId": "M101671",
                "statusId": "ITEM_APPROVED",
                "statusUserLogin": "system",
                "createdStamp": 1782174500169,
                "statusDatetime": 1782174504654,
                "_entity": "org.apache.ofbiz.order.order.OrderStatus"
            }
        ],
        "riskLevelEnumId": "ORLVL_NONE",
        "orderDate": 1782174464000,
        "riskRecommendationEnumId": "ORREC_NONE"
    }
]
```
New API payload (resp.data[0]) - exact keys and their direct mapping  (without  || fallback ):
| **UI Feature / Need** | **Old Payload**         | **New Payload (`resp.data[0]`)**      | **Direct Mapping**               |
| --------------------- | ----------------------- | ------------------------------------- | -------------------------------- |
| Response Container    | `resp.data.orderDetail` | `resp.data[0]`                        | `const data = resp.data[0];`     |
| Status ID             | `orderStatusId`         | `statusId`                            | `data.statusId`                  |
| Customer ID           | `partyId`               | `billToPartyId`                       | `data.billToPartyId`             |
| Shopify Order ID      | `orderExternalId`       | `externalId`                          | `data.externalId`                |
| Customer Name         | `customerFirstName`     | `roles[] → BILL_TO_CUSTOMER → person` | Extract from `data.roles`        |
| Billing Address       | `billingAddress`        | `contactMechs[] → POSTAL_ADDRESS`     | Extract from `data.contactMechs` |
| Email                 | `billingEmail`          | `contactMechs[] → EMAIL_ADDRESS`      | Extract from `data.contactMechs` |
| Phone                 | `billingPhone`          | `contactMechs[] → TELECOM_NUMBER`     | Extract from `data.contactMechs` |
| Approved Date         | `statusDatetime`        | `statuses[] → createdStamp`           | Extract from `data.statuses`     |

### Got a new finding: why are we showing a stale customer record on the frontend?
- We can see in earlier payload we were getting customer role directly after view entity has arranged that for us via a service but now, when we use a direct and automatic entity facade response, we see that we get everything and need to apply filtering from our end that is frontend.
- We can get data from backend in several ways like through data documents, service(view entity), REST entity //[check perform find usage - reference]. When to use which one of them, what is the ideal case and best practice? 
--------



