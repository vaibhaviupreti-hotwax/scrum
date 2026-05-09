## Header
=========
# Product (1)
# ProductType (2)
# GoodIdentification (3)
# GoodIdentificationType
# ProductAssoc (5) - Includes MARKETING pkg handling (Bundles/Kits)
# ProductAssocType (6)

## Price
========
# ProductPrice (4)
# ProductPricePurpose
# ProductPaymentMethodType

## Feature
==========
# ProductFeature (10)
# ProductFeatureType (11)
# ProductFeatureAppl (12)
# ProductFeatureApplType 
# ProductFeatureIactn 
# ProductFeatureIactnType

## Category
===========
# ProductCategory (7) - Includes ProductCategoryMember
# ProductCategoryType (8)
# ProductCatalog (9)
# ProductCatalogTypes
# ProductCatalogCatagory
# ProductCatalogCatagoryRollUp

---

## [ ] KIT AND BUNDLE CASE
## [ ] PRODUCT FEATURES

---

# Understanding the Product Data Model (A Beginner's Guide)

Think of the **Product** entity as the "Sun" in our solar system—everything else (Prices, Features, Identifications) orbits around it. This model is based on Apache OFBiz, and it’s built to be super flexible.

## 1. Core Foundation: The "Sun" of our System

| If you want to know... | Look in this Entity... | Analogy |
| --------------------- | ---------------------- | ------- |
| "What is it?"         | `Product`              | The person |
| "What are its IDs?"   | `GoodIdentification`   | Their ID card |
| "Who are its kids?"   | `ProductAssoc`         | Family tree |
| "What color is it?"   | `ProductFeature`       | Hair/Eye color |
| "How much is it?"     | `ProductPrice`         | Ticket price |
| "Where does it sit?"  | `ProductCategory`      | The group they belong to |

### The Two Most Important Types:
- **VIRTUAL_PRODUCT**: This is a "Template" or a "Parent". You can't sell a "Classic Cotton Tee" by itself—you need to know the size and color first.
- **FINISHED_GOOD**: This is the actual physical item you can touch and ship (e.g., "Cotton Crewneck - Blue - Large").
- **SERVICE**: Something like "Gift Wrapping" or "Consultation".

---

## 2. GoodIdentification: The Product's "Nicknames"

HotWax has its own internal ID (like `14730`), but Shopify or NetSuite don't know that ID. They need a "Nickname".

- **What it Stores**: Pairs of **ID Type** and **ID Value** (e.g., Type: `SKU`, Value: `POLLO-BLACK-S`).
- **How it Works**: One product can have many nicknames! It might have a `SKU` for the warehouse, a `UPCA` for the store, and a `SHOPIFY_ID` for the website.

### Why the `thruDate`? (Temporal Versioning)

In HotWax Commerce, we rarely delete records. Instead, we use "Temporal Versioning":

- **Historical Accuracy**: By setting a `thruDate`, we keep a record of what the SKU or Price was in the past. This is vital for auditing and reporting (e.g., *"What was the SKU of this product when the order was placed 6 months ago?"*).
- **Updates vs. Deletes**: When a value changes (like your SKU update from `Pollo-SNEW` to `pollo-s-black`), the old record is **"thrud" (expired)** and a new record is created with a new `fromDate`.

**Active Record Rule**: When you query these tables, always filter for the active record:
`WHERE (thru_date IS NULL OR thru_date > NOW()) AND from_date <= NOW()`

---

## 3. Relationships: How are Products connected?

### **ProductAssoc (Product Association)**
This is the "Link" between products.
- **PRODUCT_VARIANT**: Connects a **Virtual** (Parent) to its **Variants** (Children).
- **PRODUCT_COMPONENT**: Used for **Kits and Bundles**. 
  - **Marketing Packages**: If you sell a "Summer Kit" that contains a Shirt and a Hat, the "Summer Kit" is the parent, and the others are components.
  - **DM Handling**: In many cases, bundles are handled via the Data Manager (DM) to manage inventory across components.

---

## 4. Pricing: How much does it cost?

### **ProductPrice**
A product can have multiple prices at once!
- **DEFAULT_PRICE**: Standard selling price.
- **LIST_PRICE**: MSRP (Manufacturer's Suggested Retail Price).
- **PROMO_PRICE**: A special sale price.

**Pro-Tip**: You can "schedule" a sale using `thruDates`. Add a `PROMO_PRICE` today with a `fromDate` for next week. The system handles the start and end automatically!

---

## 5. Features: Attributes like Color & Size

- **ProductFeature**: The actual value (e.g., `Red`, `Small`).
- **ProductFeatureAppl (Application)**: Links the feature to the product.
  - **STANDARD_FEATURE**: The product *is* this (e.g., a Black Shirt).
  - **SELECTABLE_FEATURE**: The customer *picks* this on a Virtual product.
- **Interactions (Iactn)**: Rules that prevent impossible combos (e.g., "If Material is Silk, Machine Wash is not allowed").

---

## 6. The Sync Flow: Putting it all together

When you sync a product to Shopify:
1. **Trigger**: Mark it "Ready for Sync".
2. **Find Variants**: Use `ProductAssoc` type `PRODUCT_VARIANT`.
3. **Gather Data**: Get SKU from `GoodIdentification`, Features from `ProductFeatureAppl`, and Price from `ProductPrice`.
4. **The Handshake**: Transform to Shopify JSON and send. 
5. **Success**: Store the returned `shopify_product_id` back into `GoodIdentification`.

---

## 7. The Explorer's Roadmap: SQL Guide

### Step 1: Understand the Types
```sql
SELECT DISTINCT PRODUCT_TYPE_ID FROM PRODUCT;
SELECT DISTINCT GOOD_IDENTIFICATION_TYPE_ID FROM GOOD_IDENTIFICATION;
```

### Step 2: Find a Parent and its Kids
```sql
-- Find a Virtual Parent
SELECT PRODUCT_ID, PRODUCT_NAME FROM PRODUCT WHERE PRODUCT_TYPE_ID = 'VIRTUAL_PRODUCT' LIMIT 5;

-- Find its Variants
SELECT PRODUCT_ID_TO, PRODUCT_ASSOC_TYPE_ID 
FROM PRODUCT_ASSOC 
WHERE PRODUCT_ID = 'YOUR_PARENT_ID' 
AND PRODUCT_ASSOC_TYPE_ID = 'PRODUCT_VARIANT';
```