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

## DEEP DIVE: CORE ENTITIES

### Entity 1: `Product` — The Anchor
The `Product` entity is the anchor of this entire data model. Everything else either describes it, organizes it, or tracks its stock.

#### Internal Name vs. Product Name
Understanding the difference between these two naming fields is critical for operations:

- **Internal Name (`internalName`)**: This is the **warehouse/operations name**. It is what your internal team uses to identify the product inside the system. These are often technical and "ugly" (e.g., using SKU codes or handles).
- **Product Name (`productName`)**: This is the **customer-facing name**. This is what appears on the website, the invoice, and the packing slip. It is meant to be clean, readable, and professional.

#### Hierarchy & Relationships
- **A Variant always has a parent product.**
- **A Virtual product never has a parent.**
- **A Standalone product has no parent and no children.**

| IS_VIRTUAL | IS_VARIANT | Description |
| :---: | :---: | :--- |
| **Y** | **N** | ✅ **Virtual Parent**: The top of the hierarchy. |
| **N** | **Y** | ✅ **Standard Variant**: A child product linked to its parent. |
| **Y** | **Y** | ✅ **Mid-level Product**: Acts as both a parent and a child. |
| **N** | **N** | ✅ **Standalone**: An independent product with no relationships. |
| **N** | **Y** (Orphan) | 🚨 **Orphan Variant**: Claims to be a variant but lacks a parent link. |

---

### Entity 2: `ProductType` — The Classifier
The `ProductType` defines the categories or behaviors of the product data.

#### The `HAS_TABLE` Pattern
Every product type—including `FINISHED_GOOD`, `MARKETING_PKG_PICK`, and `DIGITAL_GOOD`—stores all its necessary data directly inside the main **`PRODUCT`** table (`HAS_TABLE = 'N'`).
- **Rationale**: None of these types require specialized fields that aren't already available in the standard `Product` entity.

#### Physical vs. Digital Behavior
The combination of `isPhysical` and `isDigital` flags determines whether an item requires warehouse operations.

| Product Type | `isPhysical` | `isDigital` | Behavior |
| :--- | :---: | :---: | :--- |
| **FINISHED_GOOD** | **Y** | **N** | Requires warehouse picking + carrier shipping. |
| **SERVICE** | **N** | **N** | No shipping; completes upon order payment. |
| **DIGITAL_GOOD** | **N** | **Y** | No shipping; triggers automated download/email. |
| **ASSET_USAGE** | **Y** | **N** | Rental logic (expects the item to be returned). |

---

### Entity 3: `ProductAssoc` — The Connective Tissue
`ProductAssoc` defines how products relate to each other. If the `Product` table is a list of individuals, `ProductAssoc` is the **Family Tree**.

- **Structure**: `productId` (Source) → `productIdTo` (Target) via `productAssocTypeId` (Verb).
- **The "Big Three" Association Types**:
    - **`PRODUCT_VARIANT`**: Links a parent "header" (T-Shirt) to its sellable SKUs (Red, Small).
    - **`PRODUCT_COMPONENT`**: Defines the "Recipe" for a kit (e.g., 1 Mat + 2 Blocks).
    - **`ALSO_BOUGHT`**: Powers website recommendations.

#### Universal Junction Table
HotWax uses `ProductAssoc` as a **Universal Junction Table**. Instead of creating separate physical tables for every relationship type, one flexible table handles everything using columns like `QUANTITY` and `SEQUENCE_NUM`.

---

### Entity 4: `GoodIdentification` — The Identifier Bridge
The `GoodIdentification` table allows you to link one internal `productId` (like `10001`) to many external codes (SKUs, Barcodes, Shopify IDs).

#### Many-to-One Identification
| `productId` | `goodIdentificationTypeId` | `idValue` | Who uses this? |
| :--- | :--- | :--- | :--- |
| `10001` | **`SKU`** | `TSHIRT-RED-L` | The Warehouse |
| `10001` | **`UPCA`** | `885001234567` | The Barcode Scanner |
| `10001` | **`EAN`** | `5012345678901` | European Retailers |
| `10001` | **`HS_CODE`** | `6109.10.00` | International Customs |
| `10001` | **`SHOPIFY_ID`** | `gid://shopify/Product/123` | The Shopify Sync |

#### Architectural Significance
- **Decoupling**: Changing a SKU during re-branding only requires updating the `GoodIdentification` record, preserving internal `productId` history.
- **Search Performance**: Barcode scans perform a high-speed lookup on the `idValue` index rather than searching the main `Product` table.
- **Composite Primary Key**: `[productId, goodIdentificationTypeId]`. This ensures one product has only one value per identification type (e.g., one SKU per product).

---

## MENTAL MAP: READING IN CLUSTERS
To understand the full architecture, read the entities in functional clusters:

1.  **Product Identity**: `ProductType` → `Product` → `GoodIdentification`.
2.  **Product Relationships**: `Product` → `ProductAssoc`.
3.  **Features & Attributes**: `ProductFeature` → `ProductFeatureAppl`.
4.  **Store & Shopify Visibility**: `ProductStore` → `ShopifyShop` → `ShopifyShopProduct`.
5.  **Inventory & Stock**: `Product` → `InventoryItem` → `InventoryItemDetail`.

---

## DATA QUALITY & DIAGNOSTICS

### Shipping Weight Audit
Missing weight on physical variants leads to shipping rate errors and carrier rejections.
```sql
SELECT p.PRODUCT_ID, p.INTERNAL_NAME, p.WEIGHT_UOM_ID, p.SHIPPING_WEIGHT
FROM PRODUCT p
WHERE p.IS_VARIANT = 'Y' AND p.WEIGHT_UOM_ID IS NULL AND p.PRODUCT_TYPE_ID = 'FINISHED_GOOD';
```

### Orphan Variant Detection
Orphan variants claim to be children but lack a parent link in `PRODUCT_ASSOC`.
```sql
SELECT p.PRODUCT_ID, p.INTERNAL_NAME
FROM PRODUCT p
WHERE p.IS_VARIANT = 'Y'
AND NOT EXISTS (SELECT 1 FROM PRODUCT_ASSOC pa WHERE pa.PRODUCT_ID_TO = p.PRODUCT_ID AND pa.PRODUCT_ASSOC_TYPE_ID = 'PRODUCT_VARIANT');
```

---

## TECHNICAL Q&A
- **Q: Why is `fromDate` in the `ProductAssoc` PK?**
    - It allows for **Relationship History**. You can link Product A to Product B multiple times across time with different quantities.
- **Q: When is weight managed at the parent level?**
    - Best when all variants weigh the same. For different weights, manage at the child level.



---
To study the data effectively, you should follow the **Dependency Sequence**. In database terms, you study the ## ROADMAP: STUDY & OPERATIONS SEQUENCE

To study the HotWax data model effectively, you should follow the **Dependency Sequence**. In database terms, this means studying the tables that "stand alone" (Primary Anchors) first, and then moving to the tables that "point" to them (Foreign Keys).

### Part 1: The Learning Sequence (Recommended Study Order)
If you are querying the database to understand the system architecture, follow this order:

1.  **The Rulebooks (Type Tables)**: 
    *   `ProductType`, `ProductAssocType`, `GoodIdentificationType`.
    *   *Why?* You must understand the categories (e.g., "What is a Finished Good?") before looking at actual products.
2.  **The Anchor**: 
    *   `Product`.
    *   *Why?* Almost every other table in the system relies on a `productId` pointing here.
3.  **The Identities**: 
    *   `GoodIdentification`.
    *   *Why?* This is the "Bridge" that allows you to find recognizable SKUs for internal Database IDs.
4.  **The Structure**: 
    *   `ProductAssoc`.
    *   *Why?* Explains how the products you've identified are grouped into Parents, Children, and Bundles.
5.  **The Warehouse Layer**: 
    *   `Facility`, `InventoryItem`, `ProductFacility`.
    *   *Why?* Adds the "Physical" layer—how many units exist and where they are located.
6.  **The Integration Layer**: 
    *   `ShopifyShop`, `ShopifyShopProduct`.
    *   *Why?* The final layer connecting internal OMS data to the external Shopify world.

---

### Part 2: The Data Entry Sequence (Sync Flow)
When a Product Sync occurs, the system populates entities in this specific order to maintain **Referential Integrity** and avoid Foreign Key errors:

| Order | Entity | Action | Architectural Reason |
| :--- | :--- | :--- | :--- |
| **1** | **`Product`** (Virtual) | Insert/Update | The "Parent" must exist before children or associations. |
| **2** | **`Product`** (Variant) | Insert/Update | Create the physical, sellable SKU unit. |
| **3** | **`ProductAssoc`** | Insert | Link Variant to Virtual (requires both IDs to exist). |
| **4** | **`GoodIdentification`** | Insert/Update | Attach the SKU/Barcode to the Variant. |
| **5** | **`ProductFeatureAppl`** | Insert | Apply Size/Color attributes to the Variant. |
| **6** | **`ProductCategoryMember`** | Insert | Place the product into the appropriate Catalog/Category. |
| **7** | **`ShopifyShopProduct`** | Insert/Update | **Critical**: Map the HotWax ID to the Shopify ID. |
| **8** | **`ProductPrice`** | Insert/Update | Set the selling price (requires the product to exist). |
| **9** | **`InventoryItem`** | Insert/Update | Initialize stock levels at the facility. |

---

### Part 3: Sequence for Operational Workflows

#### A. Inventory Adjustment (Manual)
1.  **`InventoryItem`**: Update the `quantityOnHandTotal`.
2.  **`InventoryItemDetail`**: **Mandatory Step**. Every change in stock must record a "Reason" (e.g., Damaged, Cycle Count) here for auditing.
3.  **`ProductInventoryItem`**: (View) used to verify the final recalculated totals.

#### B. Order Fulfillment (The "Pick" Flow)
1.  **`PicklistItem`**: Identifies which specific product is being requested from the shelf.
2.  **`InventoryItem`**: Validates the `AVAILABLE` status of the unit.
3.  **`ItemIssuance`**: Links the specific physical unit (`InventoryItem`) to the `OrderItem`.
4.  **`InventoryItemDetail`**: Finalizes the transaction, recording that the item has left the warehouse.

> [!TIP]
> **The Golden Rule of Study**: Always start with **`Product`**. If you don't understand the `productId`, every other table will be confusing. Once you have the ID, move to **`GoodIdentification`** to translate it into a recognizable SKU.