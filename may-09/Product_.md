y multiple products | how to use data model | kit and bundle case | product features | table - product , product_type , good_idenf and type , product_att , product_assoc , product_assoc_type , product_category , product_cate_mem , product_store_catelog , product_store_cetelog_type , product_feature , product_feature_type , product_feature_appl , product_feature_appl_type | real world use-case using | Chapter-4 products. 

# Product Domain Study Guide (Chapter 4)
*Focusing on Bundles, Kits, and Advanced Product Modeling*

## 1. Core Product Definition
*   **Entities**: `Product`, `ProductType`, `GoodIdentification`, `GoodIdentificationType`
*   **Goal**: Master how to use the data model to define basic items vs complex items.

## 2. Bundles, Kits & Marketing Packages
*   **Kit vs. Bundle**: Understand the distinction between a pre-packaged Kit and a logical Bundle.
*   **Marketing Package Handling**: How the system processes promotional packages.
*   **Record Distinction**: Understanding why a Bundle and its constituent items are treated as separate records in the `Product` table (e.g., 2+ records).
*   **Inventory Logic**: "Same inventory, multiple products" — how the system tracks inventory for a kit where the components are also sold individually.
*   **SKUs**: Managing SKUs at the Bundle level vs the individual Product level.

## 3. Relationships & Hierarchy (The "Glue")
*   **Entities**: `ProductAssoc`, `ProductAssocType`
*   **Concept**: This is the core table that links components to a Bundle or Variants to a Parent.

## 4. Product Attributes & Features
*   **Entities**: 
    *   `ProductAttribute` (Static attributes)
    *   `ProductFeature`, `ProductFeatureType` (Color, Size, etc.)
    *   `ProductFeatureAppl`, `ProductFeatureApplType` (How features are linked to products)

## 5. Pricing & Categorization
*   **Pricing**: `ProductPrice`
*   **Grouping**:
    *   `ProductCategory`, `ProductCategoryMember`
    *   `ProductStoreCatalog`, `ProductStoreCatalogType` (Linking products to specific web or store catalogs)

## 6. Real-World Application
*   [ ] Research real-world use-cases for Kit/Bundle fulfillment.
*   [ ] Trace how a "Marketing Package" is broken down into component shipments.
