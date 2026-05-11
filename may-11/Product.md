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
| `ProductFeature`      | `ProductFeatureType`         |
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