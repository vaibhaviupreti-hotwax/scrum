# Product Study Topics - May 11

## Analogies Used
- **Yoga Kit Analogy**: Used to explain how Native Management and Fulfillment Visibility work (e.g., seeing individual items on a picking list).
- **"Weakest Link" Principle**: Explained the LCD Inventory logic using a Mat, Block, and Strap example (you can only sell as many kits as your rarest item).
- **Water Bottle Example**: Clarified the 2 Records Pattern—why you need separate records for the same physical item when it's sold alone vs. in a bundle.
- **Road Map Analogy**: Simplified the Data Model relationship (Product as the destination, ProductAssoc as the road).

## 1. Product Types & Structures (Bundles/Kits)
- **Bundle & Kit Products**: Imagine a "Yoga Starter Kit" containing a Mat, a Block, and a Strap.
    - **Native Management**: Unlike some systems that need a separate "Bundle App," HotWax knows internally that the "Yoga Kit SKU" is just a collection of other SKUs.
    - **Fulfillment Visibility**: When a customer buys the kit, the warehouse worker sees three separate items on their picking list (Mat, Block, Strap) rather than one mysterious "Kit" box. This ensures nothing is forgotten.
    - **"Kit and Bundle Case"**: Think of a "Summer Look" set. It's one click for the customer, but multiple items for the inventory.
- **Market Package Handling**: 
    - **Concept**: A "Market Package" (`MARKET_PKG`) is often used when items are grouped for marketing (e.g., "Buy 3 and Save").
    - **Component Breakdown**: As soon as the order is placed, the system automatically breaks down that one line item into its components. This ensures that inventory for each individual item is reserved immediately and provides **Fulfillment Visibility** for the warehouse team.
- **Data Model Implementation (2 Records Pattern)**: 
    - **Why?**: You might sell a "Water Bottle" by itself (Record 1) and also include it in a "Hiker's Bundle" (Record 2). 
    - Even though they share the same physical bottle, they need different "Product" records in the database to have different descriptions, photos, and prices.
- **SKU Management**: 
    - **ProductAssoc**: This is the "glue" entity. It tells the system: "Product A is a component of Product B." 

## 2. Product Attributes & Configuration
- **Product Features**: 
    - **Variants**: These are the specific choices like "Red," "Blue," "Small," or "Large."
    - **In Bundles**: You could have a "Workout Bundle" where the user chooses their size. The system uses `product_feature` to map the specific "Large Shirt" to that bundle instance.

## 3. Pricing & Inventory
- **Pricing**: 
    - **Fixed**: The "Yoga Kit" costs $50, regardless of the individual item prices.
    - **Calculated**: The kit price is simply the sum of the Mat ($20) + Block ($10) + Strap ($5) = $35.
- **Inventory Logic (Lowest Common Denominator - LCD)**: 
    - **The "Weakest Link" Principle**: If you have 100 Mats, 50 Blocks, but only **2 Straps**, you can only sell **2 Yoga Kits**. 
    - The inventory of the bundle is always equal to the component with the *lowest* stock.
- **Location-Specific ATP (Available to Promise)**: 
    - If Store A has the Mat and Block, but the Strap is only in Store B, Store A cannot fulfill the bundle. HotWax checks availability *per location* to ensure a customer gets their whole kit from one place.

## 4. General
- **Data Model Mastery**: 
    - Think of the data model as a map: `Product` is the destination, `ProductAssoc` is the road between them, and `ProductPrice`/`Feature` are the details you find when you get there.

## Entity Quick Reference

| Concern                         | Stored In                |
| :------------------------------ | :----------------------- |
| **Core product**                | `Product`                |
| **Product type**                | `ProductType`            |
| **SKU/barcode**                 | `GoodIdentification`     |
| **Parent-child relation**       | `ProductAssoc`           |
| **Features like color/size**    | `ProductFeature`         |
| **Applying feature to product** | `ProductFeatureAppl`     |
| **Categories**                  | `ProductCategoryMember`  |
| **Pricing**                     | `ProductPrice`           |
| **Store visibility**            | `ProductStoreCatalog`    |

---

## PRODUCT SYNC FLOW — ENTITY SEQUENCE

### INPUT
- **Shopify Webhook Payload**: contains `shopifyProductId`, `variants`, `SKU`, `price`.

---

### EXECUTION STEPS

**STEP 1 — Resolve store context**
- **Entity**: `ShopifyShop`
- **Action**: Get `productStoreId`.

**STEP 2 — Load store rules**
- **Entity**: `ProductStore`
- **Action**: Retrieve currency, catalog, and configuration rules.

**STEP 3 — Mapping check**
- **Entity**: `ShopifyShopProduct`
- **Action**: Query by `shopifyProductId`.
    - **[FOUND]**: Retrieve existing `productId` and skip to Step 5.
    - **[NOT FOUND]**: Proceed to Step 4.

**STEP 4 — Fallback: SKU lookup**
- **Entity**: `GoodIdentification`
- **Action**: Match by SKU / barcode (`idValue`). 
    - If found, retrieve `productId`. 
    - If still not found, begin **CREATE** flow.

**STEP 5 — Create / update virtual product**
- **Entity**: `Product` (`isVirtual = Y`)
- **Action**: Update/Create fields: title, handle, status, internalName.

**STEP 6 — Variant mapping**
- **Entity**: `Product` (`isVariant = Y`)
- **Entity**: `ProductAssoc` (`productAssocTypeId = PRODUCT_VARIANT`)
- **Action**: Links virtual parent to each variant child.

**STEP 7 — Bundle check**
- **Entity**: `ProductType` (`productTypeId = MARKETING_PKG_PICK`)
- **Entity**: `ProductAssoc` (`productAssocTypeId = PRODUCT_COMPONENT`)
- **Action**: Link components and set the `quantity` field (units per component).

**STEP 8 — Write identifiers**
- **Entity**: `GoodIdentification`
- **Action**: Write SKU, UPCA, or barcode per variant (`goodIdentificationTypeId = SKU / UPCA / EAN`).

**STEP 10 — Assign features (options)**
- **Entity**: `ProductFeature` (Values: Red, Small, Nike, etc.)
- **Entity**: `ProductFeatureAppl` (`productFeatureApplTypeId = SELECTABLE_FEATURE`)
- **Action**: Link features to the variant product.

**STEP 11 — Categorize product**
- **Entity**: `ProductCategoryMember`
- **Action**: Link `productId` to `productCategoryId` (with `fromDate` / `thruDate`).

**STEP 12 — Store / catalog visibility**
- **Entity**: `ProductStoreCatalog`
- **Entity**: `ProdCatalogCategory` (`prodCatalogCategoryTypeId = PCCT_BROWSE_ROOT / PCCT_SEARCH`)
- **Action**: Link `productStoreId` to `prodCatalogId`.

**STEP 13 — Finalize mapping**
- **Entity**: `ShopifyShopProduct`
- **Action**: Save `shopifyInventoryItemId` and create the permanent link: `productId` ↔ `shopifyProductId`.

**STEP 14 — Inventory record**
- **Entity**: `InventoryItem` (`quantityOnHandTotal`, `availableToPromiseTotal`)
- **Entity**: `ProductFacility`
- **Action**: Link `productId` to `facilityId` (warehouse).

**STEP 15 — Bundle LCD calculation (if bundle)**
- **Entity**: `InventoryItemDetail`
- **Action**: For each component in `ProductAssoc`:
    - `bundleQty = min(componentQty / requiredQty)`
    - Follows the **"Weakest Link"** / Lowest Common Denominator principle.

**STEP 16 — Location activation**
- **Entity**: `ShopifyProdLocActivation`
- **Action**: Records `shopifyLocationId` and `activated` flag for the variant.

---

### OUTPUT
- **Sync Complete**: Product is fully mapped, priced, categorized, inventory is linked, and locations are activated.

---

## TECHNICAL INSIGHTS & ARCHITECTURAL DETAILS

### Key Safety & Logic Patterns
1.  **SKU Fallback (Step 4)**: This is a critical safety mechanism. If a `ShopifyProductId` changes but the SKU remains the same, this lookup prevents the creation of duplicate products in the OMS.
2.  **Bundle Architecture (Steps 7 & 15)**: The system maintains a clear distinction between the **Definition** (`ProductAssoc`) and the **Calculation** (LCD "Weakest Link"). The `getMktgPackagesAvailable` service uses the association to perform the real-time calculation.
3.  **Location Activation (Step 16)**: The `ShopifyProdLocActivation` entity (found in the `shopify-connector`) ensures that inventory is only pushed to Shopify locations that have been explicitly activated in the OMS, allowing for granular multi-location control.

### Implementation Note on Inventory
- **Step 15 Performance**: In production, the "Weakest Link" calculation primarily reads from **`InventoryItem`** (current totals) for speed. While **`InventoryItemDetail`** (transaction history) is the source of truth, it is typically only used if the system needs to perform a full re-count or audit from scratch.

---

## Data Model Design Pattern: Main & Type Tables

A key pattern in the HotWax/OFBiz data model is that most "Main" entity tables have a corresponding "Type" table. The Main table stores the actual data, while the Type table defines the categories or behaviors of that data.

| Main Table (Data)    | Type Table (Definition)      |
| :------------------- | :--------------------------- |
| `Product`            | `ProductType`                |
| `ProductAssoc`       | `ProductAssocType`           |
| `GoodIdentification` | `GoodIdentificationType`     |
| `ProductFeature`     | `ProductFeatureType`         |
| `ProductFeatureAppl` | `ProductFeatureApplType`     |

---

Product is the anchor of this entire data model. Everything else either:
- **Describes** the product (`GoodIdentification`, `ProductAttribute`, `ProductFeature`)
- **Organizes** the product (`ProductCategory`, `ProductCategoryMember`)
- **Connects** the product to Shopify (`ShopifyShopProduct`)
- **Tracks** the product's stock (`InventoryItem`, `ProductFacility`)

---

## Understanding Type Tables
The **Type table** serves as a predefined lookup list. It defines the allowed values and behaviors for a corresponding Main table field. 

- **The Dropdown Analogy**: Think of it like a dropdown menu in a form—the Type table defines the options available in that menu.
- **Example**: `ProductAssocType` contains values like `PRODUCT_VARIANT`, `PRODUCT_COMPONENT`, and `ALSO_BOUGHT`. The `ProductAssoc` table then uses these values to define the specific relationship between two products.
- **Rule of Thumb**: You **never** store dynamic business data in Type tables. They exist solely to define the rules and categories of the system.

---

## Mental Map: Reading the Data Model in Clusters

To understand the full architecture, it's easier to read the entities in **Clusters** rather than all at once. Here is a breakdown of the five functional clusters:

### Cluster 1 — Product Identity (The Core)
`ProductType` → `Product` → `GoodIdentification` → `GoodIdentificationType`
- **Focus**: "What is this product and what are its unique identifiers (SKUs/Barcodes)?"

### Cluster 2 — Product Relationships
`Product` → `ProductAssoc` ← `ProductAssocType`
- **Focus**: "How are products connected to each other (e.g., Parent/Child, Bundle/Component)?"

### Cluster 3 — Features and Attributes
`ProductFeatureType` → `ProductFeature` → `ProductFeatureAppl` ← `ProductFeatureApplType`
- **Focus**: "What options does this product have (e.g., Size, Color, Brand)?"

### Cluster 4 — Store & Shopify Visibility
`ProductStore` → `ShopifyShop` → `ShopifyShopProduct` → `ShopifyProdLocActivation`
- **Focus**: "Which store is this product in and how is it mapped to the Shopify environment?"

### Cluster 5 — Inventory & Stock Tracking
`Product` → `InventoryItem` → `InventoryItemDetail` + `ProductFacility`
- **Focus**: "How much stock exists, where is it located, and what specific transactions changed the count?"

---

## Real-World Scenario: Syncing a "Red Small Cotton T-Shirt"

Here is how the sync flow looks when a specific product moves from Shopify to the OMS:

1.  **ShopifyShop**: The system identifies which specific Shopify store sent the webhook.
2.  **ProductStore**: It loads the rules for that store (e.g., USD currency, "Main" catalog).
3.  **ShopifyShopProduct**: Checks if `shopifyProductId` already exists. 
    - *Result*: If **Yes**, it gets the `productId`. If **No**, it moves to lookup.
4.  **GoodIdentification**: Looks up the SKU to see if this product exists in the OMS (even if not yet linked to Shopify).
5.  **Product (Virtual)**: Creates or updates the parent "Cotton T-Shirt" record (`isVirtual=Y`).
6.  **Product (Variant) + ProductAssoc**: Creates the "Red Small" child record (`isVariant=Y`) and links it to the parent using `PRODUCT_VARIANT`.
7.  **GoodIdentification**: Saves the specific SKU and barcode for this new variant.
8.  **ProductFeature + ProductFeatureAppl**: Saves "Red" and "Small" as features and applies them to the variant.
9.  **ProductCategoryMember**: Places the product into the "Men's Apparel" category.
10. **ShopifyShopProduct**: Saves the final, permanent mapping between `productId` and `shopifyProductId`.
11. **InventoryItem + ProductFacility**: Links stock levels to the physical warehouse facility.
12. **ShopifyProdLocActivation**: Marks which specific Shopify locations this variant should be active in.

> [!NOTE]
> Every one of these steps corresponds to a "box" in the architectural diagram. The lines in the diagram show exactly how data is handed off from one entity to the next.

---

## Data Quality Audit: Shipping Weights

### The "Null Weight" Problem
A common data quality issue occurs when variants (physical products) are missing weight information. 

**Detection Query:**
```sql
SELECT
    p.PRODUCT_ID,
    p.PRODUCT_TYPE_ID,
    p.INTERNAL_NAME,
    p.WEIGHT_UOM_ID,
    p.SHIPPING_WEIGHT,
    parent.PRODUCT_ID        AS PARENT_ID,
    parent.WEIGHT_UOM_ID     AS PARENT_WEIGHT_UOM,
    parent.SHIPPING_WEIGHT   AS PARENT_SHIPPING_WEIGHT
FROM PRODUCT p
LEFT JOIN PRODUCT_ASSOC pa
    ON pa.PRODUCT_ID_TO = p.PRODUCT_ID
    AND pa.PRODUCT_ASSOC_TYPE_ID = 'PRODUCT_VARIANT'
    AND pa.THRU_DATE IS NULL
LEFT JOIN PRODUCT parent
    ON parent.PRODUCT_ID = pa.PRODUCT_ID
WHERE p.IS_VARIANT = 'Y'
    AND p.WEIGHT_UOM_ID IS NULL
    AND p.PRODUCT_TYPE_ID = 'FINISHED_GOOD'
LIMIT 5;
```

### Risks of Missing Weight Data
If `WEIGHT_UOM_ID` is NULL and `SHIPPING_WEIGHT` is 0 (with no parent fallback), it can lead to:
- **Incorrect Shipping Rates**: Customers might be undercharged for shipping.
- **Carrier Rejections**: Some carrier APIs will reject shipments with zero weight.
- **Insurance & Compliance**: High-value items (like jewelry) require accurate weight for declared value and insurance.
- **Silent Failures**: The system might default to a flat rate, masking the data issue without alerting anyone.

---

## Product Hierarchy & Relationships

### Core Definitions
- **A Variant always has a parent product.**
- **A Virtual product never has a parent.**
- **A Standalone product has no parent and no children.**

The `PRODUCT_ASSOC` table is the "truth" for these links. If a product isn't listed as a `PRODUCT_ID_TO` in a `PRODUCT_VARIANT` association, it has no parent.

### Matrix: Identifying Product States
| IS_VIRTUAL | IS_VARIANT | Description |
| :---: | :---: | :--- |
| **Y** | **N** | ✅ **Virtual Parent**: The top of the hierarchy. |
| **N** | **Y** | ✅ **Standard Variant**: A child product linked to its parent. |
| **Y** | **Y** | ✅ **Mid-level Product**: Acts as both a parent and a child. |
| **N** | **N** | ✅ **Standalone**: An independent product with no relationships. |
| **N** | **Y** (Orphan) | 🚨 **Orphan Variant**: Claims to be a variant but lacks a parent link. |

---

## Orphan Variant Detection

"Orphan variants" are products marked as `IS_VARIANT = 'Y'` that are missing a corresponding entry in the `PRODUCT_ASSOC` table.

**Detection Query:**
```sql
SELECT p.PRODUCT_ID,
       p.INTERNAL_NAME,
       p.PRODUCT_NAME
FROM PRODUCT p
WHERE p.IS_VARIANT = 'Y'
AND NOT EXISTS (
    SELECT 1
    FROM PRODUCT_ASSOC pa
    WHERE pa.PRODUCT_ID_TO = p.PRODUCT_ID
    AND pa.PRODUCT_ASSOC_TYPE_ID = 'PRODUCT_VARIANT'
);
```

### Common Causes for Orphans
| Cause | Description |
| :--- | :--- |
| **Broken Shopify Sync** | The parent was not imported or the sync failed mid-way. |
| **Parent Deletion** | The parent was removed from the system, but the variants were left behind. |
| **Incomplete Migration** | Only specific sellable SKUs were migrated without their virtual parents. |
| **Incorrect Flagging** | A product was accidentally marked as a variant during manual entry. |

---

## Technical Q&A
- **Q: When is weight managed at the parent level vs. child level?**
    - **Parent Level**: Best for products where all variants weigh the same (e.g., different colors of the same t-shirt).
    - **Child Level**: Essential when variants have different weights (e.g., a 1lb box vs. a 5lb box of the same product).
    - **Best Practice**: Always check the parent if the child's weight is NULL.

---

## Internal Name vs. Product Name

Understanding the difference between these two naming fields is critical for operations:

### 1. Internal Name (`internalName`)
This is the **warehouse/operations name**. It is what your internal team uses to identify the product inside the system. These are often technical and "ugly" (e.g., using SKU codes or handles).

**Rules:**
- **Practically Mandatory**: It should always be filled in.
- **Search-Friendly**: Must be unique enough to find the product without confusion.
- **Flexible Format**: No strict formatting rules—can be a SKU, a handle, a code, etc.
- **OMS Focused**: Used heavily in search, lookups, sync processes, and logs.

### 2. Product Name (`productName`)
This is the **customer-facing name**. This is what appears on the website, the invoice, and the packing slip. It is meant to be clean, readable, and professional for display purposes.

---

### Comparison: Internal vs. Product Name

| Feature | Internal Name | Product Name |
| :--- | :--- | :--- |
| **Audience** | Your internal team | Your customers |
| **Example** | `V_abominable-hoodie` | `Abominable Hoodie` |
| **Used in** | OMS search, sync, logs | Website, invoice, packing slip |
| **Can be "ugly"?** | Yes | No (must be professional) |
| **Always filled?** | Yes | Sometimes blank on virtuals |

---

## Primary Category vs. Category Membership

HotWax uses a dual-category system to provide both structural integrity and marketing flexibility.

> [!TIP]
> **The Analogy**: Think of it as your **Home Address** vs. **Places you visit**. 
> - `primaryProductCategoryId` is your permanent home address.
> - `ProductCategoryMember` records every place you've ever been or currently belong to.

---

### 1. `primaryProductCategoryId` (Inside the `Product` Entity)
This is a direct field on the product record. Think of it as the product's **Primary Identity**.

- **Quantity**: A product can have only one (or zero) primary category.
- **Purpose (The "Home" Category)**:
    - **Breadcrumbs**: It determines the default path shown on a website (e.g., *Home > Apparel > T-Shirts > [Product]*).
    - **Logic Defaults**: Often used to inherit tax rules or shipping settings if they aren't defined on the product itself.
    - **Reporting**: Essential for "clean" sales reports where every product belongs to exactly one department (no double-counting).
    - **Solr Faceting**: The search engine uses this as the "Main" facet for the product.

### 2. `productCategoryId` (Inside the `ProductCategoryMember` Entity)
This is a link in a separate table. It defines **Membership** or **Tagging**.

- **Quantity**: A product can have infinite memberships.
- **Purpose (Discovery & Marketing)**:
    - **Multiple Placements**: A "Red Running Shoe" can be a member of "Men's Shoes," "Running Gear," "New Arrivals," and "Sale Items" simultaneously.
    - **Seasonal Collections**: You can add products to a "Summer 2024" category using `fromDate` and `thruDate` without changing their "Home" category.
    - **Search Filters**: Powers the "Refine By" filters (e.g., Filtering by "Material: Cotton").
    - **Browsing**: When a customer clicks on "Sale Items," the system queries `ProductCategoryMember` to find all linked products.

---

### Comparison: Primary Category vs. Membership

| Feature          | `primaryProductCategoryId`                  | `ProductCategoryMember`                      |
| :--------------- | :------------------------------------------ | :------------------------------------------- |
| **Table**        | `Product`                                   | `ProductCategoryMember`                      |
| **Relationship** | 1-to-1 (Product has one Home)               | Many-to-Many (Product has many Tags)         |
| **Data Type**    | Foreign Key (pointing to `ProductCategory`) | Part of a Composite Primary Key              |
| **Timing**       | Static (Permanent)                          | Temporal (has `fromDate`/`thruDate`)         |
| **Best For**     | Breadcrumbs, Reports, Tax logic             | Navigating, Sales, Promotions                |

**Why do we need both?** 
If we only had `ProductCategoryMember`, the website wouldn't know which category is the "main" one to show in the URL. If we only had `primaryProductCategoryId`, we could never put a product in more than one category at a time. This system provides **One Home, Many Doors**.

---

## Inventory Item Types: How the System Counts

The `inventoryItemTypeId` determines the fundamental logic for how a product is tracked, counted, and fulfilled in the warehouse.

### 1. `NON_SERIAL_INV_ITEM` (Non-Serialized)
Used for identical items where you only care about the **total quantity**.

- **Analogy**: A box of 500 identical blue pens. You don't care which specific pen you pick; you just need to know there are 500 in the box.
- **Database Behavior**: 
    - The system creates **one** `InventoryItem` record for that product per warehouse/facility.
    - The `quantityOnHandTotal` field stores the bulk count (e.g., `500`).
- **Fulfillment**: The system simply subtracts `1` from the total count during shipment.
- **Typical Products**: T-shirts, water bottles, socks, groceries.

### 2. `SERIALIZED_INV_ITEM` (Serialized)
Used for high-value items where **every single unit is unique** and must be tracked individually.

- **Analogy**: iPhones. Every iPhone has a unique IMEI/Serial Number. You must know exactly which specific unit was sold for warranty and security purposes.
- **Database Behavior**: 
    - The system creates a **separate `InventoryItem` record for every single physical unit**.
    - If you have 500 iPhones, you have 500 distinct records.
    - Each record has `quantityOnHandTotal = 1` and a unique `serialNumber`.
- **Fulfillment**: You must scan the **specific serial number** during packing. The system then marks that unique record as `DELIVERED`.
- **Typical Products**: Smartphones, Laptops, Luxury Watches, Electronics.

---

### Comparison: Non-Serialized vs. Serialized

| Feature | `NON_SERIAL_INV_ITEM` | `SERIALIZED_INV_ITEM` |
| :--- | :--- | :--- |
| **Tracking Method** | By Quantity (Bulk Sum) | By Unique Serial Number |
| **Database Records** | 1 Record = Many Units | 1 Record = 1 Unit |
| **Scanning at Picking** | Scan SKU once, pick quantity | Scan every single unit's Serial # |
| **Fulfillment Logic** | `Total - 1` | Update specific record to `DELIVERED` |
| **Cost of Management** | Low (Fast processing) | High (Requires intense scanning) |
| **Best For** | Low-cost, mass-produced items | High-value, warrantied items |

> [!IMPORTANT]
> The `inventoryItemTypeId` defaults to `NON_SERIAL_INV_ITEM` for most retail products to keep operations fast. You only switch to `SERIALIZED` if the business explicitly requires individual unit tracking.

---

## Implementation Lifecycle: Where the Type Matters

The inventory type isn't just a label—it changes the behavior of the code at every step.

### 1. Product Definition (The "Master Switch")
- **Location**: Managed in the `Product` entity.
- **Implementation**: When creating a product in the HotWax UI (`CreateProduct.ftl`), the "Serialized" checkbox determines this value.
- **Code Reference**: `ProductWorker.java` contains an `isSerialized()` method that checks if the field is set to `SERIALIZED_INV_ITEM`.

### 2. Receiving (The "Point of Entry")
- **Implementation**: Handled in `ShipmentReceiptServices.xml` and the Receive Purchase Order screen (`ReceivePurchaseOrder.ftl`).
- **Logic**:
    - **Non-Serial**: UI shows a single "Quantity" box. Entering "50" creates one record with a total of 50.
    - **Serialized**: UI prompts for 50 unique serial numbers. The system creates 50 separate `InventoryItem` records, each with `quantity=1`.

### 3. Fulfillment (Picking & Packing)
- **Implementation**: `PicklistServices.xml` and `IssuanceServices.xml`.
- **Logic**:
    - **Non-Serial**: Picklist says: "Grab 2 units from Bin 5."
    - **Serialized**: Picklist says: "Grab 2 units from Bin 5 AND scan the serial numbers of the specific units picked."
- **Database**: The `ItemIssuance` entity records the specific `inventoryItemId` (which, for serialized items, links to a specific serial number).

### 4. Inventory Calculations (ATP)
- **Implementation**: `InventoryServices.java`.
- **Logic**:
    - **Non-Serial**: Reads the `availableToPromiseTotal` directly from the single summary record.
    - **Serialized**: Performs a `COUNT` of all individual `InventoryItem` records for that product with a status of `INV_AVAILABLE`.

---

### Implementation Summary: The "Where"

| Context | Implementation Point | Action based on Type |
| :--- | :--- | :--- |
| **Setup** | `Product` Entity | Sets the tracking rule for the product lifecycle. |
| **Inbound** | `ShipmentReceiptServices` | Decides whether to create 1 bulk record or many unique records. |
| **Storage** | `InventoryItem` Table | Determines if the `serialNumber` column must be populated. |
| **Outbound** | `PicklistServices` | Determines if the packer must scan a unique unit ID. |
| **UI** | `ReceiveProduct.ftl` | Switches between a "Quantity" input and a "Serial #" list. |

---

## Product Type Behavior: Physical vs. Digital

The combination of `isPhysical` and `isDigital` flags in the `ProductType` seed data determines whether an item requires warehouse operations or can be fulfilled instantly.

| Product Type | `isPhysical` | `isDigital` | Behavior |
| :--- | :---: | :---: | :--- |
| **FINISHED_GOOD** | **Y** | **N** | Requires warehouse picking + carrier shipping. |
| **SERVICE** | **N** | **N** | No shipping; completes upon order payment. |
| **DIGITAL_GOOD** | **N** | **Y** | No shipping; triggers automated download/email. |
| **ASSET_USAGE** | **Y** | **N** | Rental logic (expects the item to be returned). |

> [!NOTE]
> The `isPhysical` flag is the master switch that determines if an item needs to enter the warehouse flow or if it can be fulfilled "instantly" by the system.