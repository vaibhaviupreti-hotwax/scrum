# Product Data Model — Learning Notes

Think of this as your "Conversational Cheat Sheet" for the Product model.

---

## 1. Core Foundation: The "Sun" of our System

| Entity                   | What it actually does                             |
| ------------------------ | ------------------------------------------------- |
| `Product`                | The anchor. Every item starts here.               |
| `ProductType`            | Tells us if it's a physical item or a template.   |
| `GoodIdentification`     | The "Link" to the outside world (SKUs, Barcodes). |
| `ProductAssoc`           | The "Family Tree" (connects parents to children). |

### The Two Most Important Types:
- **VIRTUAL_PRODUCT**: This is a "Template" or a "Parent". You can't put this in a box; it's just a placeholder for its variants (e.g., "Cotton Crewneck").
- **FINISHED_GOOD**: This is the actual physical item you can touch and ship (e.g., "Cotton Crewneck - Blue - Large").

---

## 2. GoodIdentification: The Product's "Nicknames"

HotWax has its own internal ID (like `14730`), but Shopify or NetSuite don't know that ID. They need a "Nickname".

- **What it Stores**: Pairs of **ID Type** and **ID Value** (e.g., Type: `SKU`, Value: `POLLO-BLACK-S`).
- **How it Works**: One product can have many nicknames! It might have a `SKU` for the warehouse, a `UPCA` for the store, and a `SHOPIFY_ID` for the website.

### Why the `thruDate`? (Version Control)
In retail, SKUs change! If a product is re-branded, the old SKU isn't deleted—it's **"thrud" (expired)**.
- **Scenario**: SKU changes from `OLD-SKU` to `NEW-SKU`.
- **The HotWax Way**: We set the `thruDate` on `OLD-SKU` to "Now" and create a new row for `NEW-SKU`.
- **The Result**: You can still look at an order from last year and see that the product was called `OLD-SKU` back then. Historical accuracy is preserved!

---

## 3. Pricing: Not just a single number

A product can have multiple prices at once (Retail, Wholesale, Promo).
- **Pro-Tip**: Prices also use `thruDates`. This is how you "schedule" a sale. You can add a `PROMO_PRICE` today and set its `fromDate` to next Saturday and `thruDate` to next Sunday. The system will automatically start and end the sale for you!

---

## 4. Features: Attributes like Color & Size

- **ProductFeature**: The actual attribute (e.g., `Red`, `Small`).
- **Standard vs Selectable**: 
  - **Standard**: A feature the product *has* (e.g., a Black Bag).
  - **Selectable**: Something the customer *picks* on a Parent product (e.g., "What color do you want?").
- **Interactions (Iactn)**: Rules that prevent impossible combos (e.g., "If you pick Silk material, you can't pick the Machine Wash feature").

---

## 5. The Sync Flow: How it actually works

When you sync a product to Shopify:
1.  **Find the Product**: Mark it "Ready for Sync".
2.  **Find the Family**: Use `ProductAssoc` to find all its variants.
3.  **Gather the Nicknames**: Get the `SKU` from `GoodIdentification`.
4.  **Gather the Cost**: Get the `ProductPrice` (check for an active `PROMO_PRICE` first!).
5.  **Gather the DNA**: Get the Color/Size from `ProductFeatureAppl`.
6.  **The Handshake**: Send it to Shopify. If Shopify says "Success", it sends back a `SHOPIFY_ID`. You save that ID back into `GoodIdentification` so the two systems are "linked."

---

## 6. The Developer's Mental Model

| If you want to know... | Look in this Entity... | Analogy |
| --------------------- | ---------------------- | ------- |
| "What is it?"         | `Product`              | The person |
| "What are its IDs?"   | `GoodIdentification`   | Their ID card |
| "Who are its kids?"   | `ProductAssoc`         | Family tree |
| "What color is it?"   | `ProductFeature`       | Hair/Eye color |
| "How much is it?"     | `ProductPrice`         | Ticket price |
| "Where does it sit?"  | `ProductCategory`      | The group they belong to |