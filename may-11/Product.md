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

| Layer | Concern | Primary Entity |
| :--- | :--- | :--- |
| **Identity** | Core product record | `Product` |
| | Product classifications | `ProductType` |
| | SKUs, Barcodes, Shopify IDs | `GoodIdentification` |
| | Flexible metadata & flags | `ProductAttribute` |
| **Structure** | Parent-child & Bundles | `ProductAssoc` |
| **Variation** | Feature categories (Color/Size) | `ProductFeatureType` |
| | Feature values (Red/Small) | `ProductFeature` |
| | Linking features to products | `ProductFeatureAppl` |
| **Organization** | Product categories (Shelves) | `ProductCategory` |
| | Category membership | `ProductCategoryMember` |
| | Category hierarchy (Rollups) | `ProductCategoryRollup` |
| **Storefront** | Store rules & settings | `ProductStore` |
| | Product catalogs | `ProdCatalog` |
| | Store-to-Catalog mapping | `ProductStoreCatalog` |
| | Catalog-to-Category mapping | `ProdCatalogCategory` |

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
| `10001` | **`SHOPIFY_PROD_ID`** | `gid://shopify/Product/123` | The Shopify Sync |

#### The "Rulebook": `GoodIdentificationType`
Think of `GoodIdentificationType` as the **Rulebook** for naming products. It defines the **categories** of numbers allowed in the system.

- **Metadata Table**: You cannot add an ID to a product unless the type (e.g., "SKU") is first defined here.
- **Hierarchy (`parentTypeId`)**: In your system, the parent type is **`HC_GOOD_ID_TYPE`**. Identifiers like `SKU`, `UPCA`, and `ISBN` all sit under this parent to be treated as scannable IDs.
- **Shopify Specifics**: Your system uses distinct types for Shopify mapping: `SHOPIFY_PROD_ID`, `SHOPIFY_PROD_SKU`, and `SHOPIFY_PROD_UPCA`.

#### Architectural Insight: Lookup vs. Operational Data
There is a design distinction between `GoodIdentification` and `ProductAttribute`:
- **`GoodIdentification`**: Stores IDs used to **find** a product (e.g., `SHOPIFY_PROD_ID`). This table is queried heavily for lookups.
- **`ProductAttribute`**: Stores IDs used **after** a product is found (e.g., `SHOPIFY_INV_ITEM_ID`). This is operational data used for the *next* step in a process.
- **Rule of Thumb**: If the ID is used to find the product — `GoodIdentification`. If it's used after — `ProductAttribute`.

---

### Entity 5: `ProductAttribute` — The Dynamic Extension
`ProductAttribute` is the **"Blank Canvas"** table for a product, using an **EAV (Entity-Attribute-Value)** model.

- **The Structure**: `productId`, `attrName` (e.g., `fabric_care`), and `attrValue` (e.g., `Machine wash cold`).
- **Significance**: Unlike other tables, this has **no Type table**. You can literally make up any `attrName` on the fly.
- **Shopify Sync Usage**: Used for "extra" Shopify data that doesn't fit standard boxes.

| `attrName` | `attrValue` | Purpose |
| :--- | :--- | :--- |
| **`is_preorder`** | `Y` | Tells the OMS to treat this as a pre-order item. |
| **`shopify_status`** | `active` | Mirrors the product status from Shopify Admin. |
| **`sync_error_log`** | `Missing weight field` | Stores debugging info if a sync fails. |

---

### Entity 6: `ProductFeatureType` — The Rulebook for Variations
`ProductFeatureType` is the **"Question"** part of the variation system (e.g., "What color is this?").

- **The Structure**: `productFeatureTypeId` (e.g., `COLOR`), `description`, and `parentTypeId`.
- **Common Seed Records**: `COLOR`, `SIZE`, `BRAND`, `FABRIC`, `STYLE`.
- **Website Logic**: Powers the filters and dropdowns. The website queries this table to know which "Questions" to ask the customer.

### Entity 7: `ProductFeature` — The Global Dictionary
If `ProductFeatureType` is the Question, `ProductFeature` is the **Answer** (e.g., "Red").

- **The Core Purpose**: It is a **Global Dictionary**. You define "Cotton" once here and link it to thousands of products.
- **The Structure**: `productFeatureId`, `productFeatureTypeId` (FK), `description`, and `uomId`.
- **Technical Power**: The `uomId` (Unit of Measure) allows for math and sorting on features like "Length" or "Weight."

### Entity 8: `ProductFeatureApplType` — The Role
Before applying a feature, you must decide its role:
1.  **`SELECTABLE_FEATURE`**: Used on **Virtual Products** (Dropdown options).
2.  **`STANDARD_FEATURE`**: Used on **Variant Products** (The actual fact of the SKU).
3.  **`DISTINGUISHING_FEATURE`**: Used for search highlights (e.g., "Waterproof").

### Entity 9: `ProductFeatureAppl` — The Junction
This is the **"Glue"** that connects your product definitions to your variation dictionary.

- **The Structure**: `productId`, `productFeatureId`, `productFeatureApplTypeId`, and `fromDate`.
- **Feature Inheritance**: It keeps data clean by explicitly stating whether a feature is a customer choice or a manufacturing fact.
- **Analogy**: If `Product` is a **Person** and `ProductFeature` is a **Shirt**, then `ProductFeatureAppl` is the **Act of Wearing the Shirt**.

---

### Entity 10: `ProductCategory` — The Shelf
Think of `ProductCategory` as the **"Physical Shelf"** in a store.

- **The Rulebook: `ProductCategoryType`**: Defines the nature of the shelf (`CATALOG_CATEGORY`, `PROMOTIONAL_CATEGORY`, `TAX_CATEGORY`).
- **Key Fields**: `productCategoryId`, `categoryName`, `description`.

### Entity 11: `ProductCategoryMember` — The Placement
The **Junction Table** that puts a `Product` onto a `Category`.

- **Structure**: `productId`, `productCategoryId`, `sequenceNum`.
- **Many-to-Many**: A product can be a member of multiple categories (e.g., "Shoes" and "Sale Items") simultaneously.
- **Sequence Logic**: The `sequenceNum` determines which product appears **first** on the collection page.

### Entity 12: `ProductCategoryRollup` — The Hierarchy
The "Hidden" table that allows for **Sub-categories** (e.g., Men's Apparel → T-Shirts).

- **Structure**: `productCategoryId` (Child) and `parentProductCategoryId` (Parent).

| Feature | `ProductCategory` | `ProductCategoryMember` | `ProductCategoryRollup` |
| :--- | :--- | :--- | :--- |
| **Analogy** | The Shelf itself. | Putting an item on the shelf. | Nailing a small shelf onto a big shelf. |
| **Logic** | Defines the "Where". | Defines the "What's inside". | Defines the "Tree Structure". |

---

## Phase 15–18: The Storefront Layer (Catalogs & Stores)

If the **Category** is the **Shelf**, then the **Catalog** is the **Section of the Store** (e.g., "Clothing Department"), and the **ProductStore** is the **Physical Store Location** or **Website Domain** itself.

### Entity 13: `ProductStore` (The Storefront)
This is the highest level of organization. A `ProductStore` represents a single point of sale—like your Shopify website.

- **Purpose**: It holds the business rules for a store (e.g., which currency to use, which warehouse to ship from, which catalog to show).
- **Key Fields**: `productStoreId`, `storeName`, `defaultCurrencyUomId`, `inventoryFacilityId`.
- **Shopify Link**: In the Shopify Sync, the `productStoreId` is the primary context used to load all configuration rules.

### Entity 14: `ProdCatalog` (The Department)
A `ProdCatalog` is a collection of categories. It allows you to group different sections of your store into one manageable unit.

- **Purpose**: It acts as a container. You might have one "Main Catalog" for your website and a different "Internal Catalog" for your warehouse team.
- **Key Fields**: `prodCatalogId`, `catalogName`.

### Entity 15: `ProductStoreCatalog` (The Store Link)
This is the **Junction Table** that decides which **Catalog** is shown in which **Store**.

- **Structure**: `productStoreId`, `prodCatalogId`, `fromDate`, `sequenceNum`.
- **Architectural Logic**: One store can have multiple catalogs (e.g., "Seasonal Catalog" + "Permanent Catalog"). The `sequenceNum` decides which one appears first in the navigation menu.

### Entity 16: `ProdCatalogCategory` (The Catalog Link)
This is the final **Junction Table** that connects a **Category** (Shelf) to a **Catalog** (Department).

- **Structure**: `prodCatalogId`, `productCategoryId`, `prodCatalogCategoryTypeId`.
- **Types of Links**:
    - **`PCCT_BROWSE_ROOT`**: This is the "Main Menu" category. Everything inside this category will appear on the website's homepage.
    - **`PCCT_SEARCH`**: This category is used for the search bar logic.
    - **`PCCT_PROMOTIONS`**: Used for the "Featured Deals" section.

---

## ENTITY NAMING CONVENTIONS

HotWax/OFBiz follows strict naming patterns for "Add-on" or "Extension" tables. If you know the main table name (`XYZ`), you can guess its extensions:

1.  **The "Attribute" Pattern**: `XYZAttribute` (Generic Name-Value pairs for metadata).
2.  **The "Type Attribute" Pattern**: `XYZTypeAttr` (Defines which attributes are allowed for a specific type).
3.  **The "Content" Pattern**: `XYZContent` (For images, PDFs, or long-form text).
4.  **The "Assoc" Pattern**: `XYZAssoc` (For linking two entities of the same type).
5.  **The "Role" Pattern**: `XYZRole` (For linking people or companies to the entity).

---

## ROADMAP: STUDY & OPERATIONS SEQUENCE

### Part 1: The Learning Sequence (Recommended Study Order)
1.  **The Rulebooks**: Type Tables.
2.  **The Anchor**: `Product`.
3.  **The Identities**: `GoodIdentification`.
4.  **The Structure**: `ProductAssoc`.
5.  **The Warehouse Layer**: `Facility`, `InventoryItem`, `ProductFacility`.
6.  **The Integration Layer**: `ShopifyShop`, `ShopifyShopProduct`.

### Part 2: The Data Entry Sequence (Sync Flow)
| Order | Entity | Action | Architectural Reason |
| :--- | :--- | :--- | :--- |
| **1** | **`Product`** (Virtual) | Insert/Update | Parent must exist first. |
| **2** | **`Product`** (Variant) | Insert/Update | Create the physical unit. |
| **3** | **`ProductAssoc`** | Insert | Link Variant to Virtual. |
| **4** | **`GoodIdentification`** | Insert/Update | Attach SKU/Barcode. |
| **5** | **`ProductFeatureAppl`** | Insert | Apply Size/Color attributes. |
| **6** | **`ProductCategoryMember`** | Insert | Place into Catalog/Category. |
| **7** | **`ShopifyShopProduct`** | Insert/Update | **Critical**: Map HotWax ID to Shopify ID. |
| **8** | **`ProductPrice`** | Insert/Update | Set the selling price. |
| **9** | **`InventoryItem`** | Insert/Update | Initialize stock levels. |

---

## DATA QUALITY & DIAGNOSTICS

### Shipping Weight Audit
```sql
SELECT p.PRODUCT_ID, p.INTERNAL_NAME, p.WEIGHT_UOM_ID, p.SHIPPING_WEIGHT
FROM PRODUCT p
WHERE p.IS_VARIANT = 'Y' AND p.WEIGHT_UOM_ID IS NULL AND p.PRODUCT_TYPE_ID = 'FINISHED_GOOD';
```

### Orphan Variant Detection
```sql
SELECT p.PRODUCT_ID, p.INTERNAL_NAME
FROM PRODUCT p
WHERE p.IS_VARIANT = 'Y'
AND NOT EXISTS (SELECT 1 FROM PRODUCT_ASSOC pa WHERE pa.PRODUCT_ID_TO = p.PRODUCT_ID AND pa.PRODUCT_ASSOC_TYPE_ID = 'PRODUCT_VARIANT');
```

---

## TECHNICAL Q&A
- **Q: Why is `fromDate` in the `ProductAssoc` PK?**
    - It allows for **Relationship History**. You can link Product A to Product B multiple times across time.
- **Q: Attribute vs. Feature vs. Identifier?**
    - **`GoodIdentification`**: Unique ID used to find the product (SKU).
    - **`ProductFeature`**: Values a customer selects (Size, Color).
    - **`ProductAttribute`**: Descriptive metadata or system flags (Washing instructions, Pre-order flag).

> [!TIP]
> **The Golden Rule**: Always start with **`Product`**. If you don't understand the `productId`, every other table will be confusing. Move to **`GoodIdentification`** to translate IDs into recognized SKUs.