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
    - **Auto-Explosion**: As soon as the order is placed, the system "explodes" that one line item into its components so the inventory for each individual item is reserved immediately.
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

