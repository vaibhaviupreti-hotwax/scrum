# HotWax Commerce: Core Domain Objects

In HotWax/OFBiz, the system architecture is organized into **Domains**. Each domain represents a major business concept and consists of a cluster of related entities and services.

## The "Big Six" Commerce Domains
These domains cover the primary operational flow of the HotWax ecosystem.

### 1. Product Domain (The Item)
*   **Purpose**: Defines the product catalog, attributes, and relationships.
*   **Core Entities**: `Product`, `ProductType`, `GoodIdentification`, `ProductPrice`, `ProductAssoc`.

### 2. Party Domain (The Stakeholder)
*   **Purpose**: Manages all individuals and organizations involved in the system.
*   **Core Entities**: 
    *   **`Person`**: Individual human attributes (First Name, Last Name).
    *   **`PartyGroup`**: Organizations or companies (Company Name).
    *   **`PartyRole`**: Defines the role (e.g., `CUSTOMER`, `SUPPLIER`, `CARRIER`).

### 3. Order Domain (The Agreement)
*   **Purpose**: Records the contractual agreement between a buyer and seller.
*   **Core Entities**: 
    *   **`OrderHeader`**: High-level order data (ID, Date, Status).
    *   **`OrderItem`**: Specific line items (Product, Quantity, Price).
    *   **`OrderRole`**: Links parties to the order.

### 4. Inventory Domain (The Stock)
*   **Purpose**: Tracks physical stock levels and transaction history.
*   **Note**: Although technically grouped under `facility` in the code, Inventory is treated as an independent functional domain.
*   **Core Entities**: 
    *   **`InventoryItem`**: A specific batch or unit of stock.
    *   **`InventoryItemDetail`**: The audit log of every stock movement (+/-).
    *   **`InventoryItemStatus`**: Stock condition (e.g., `AVAILABLE`, `DAMAGED`).

### 5. Facility Domain (The Location)
*   **Purpose**: Defines physical and geographical storage structures.
*   **Core Entities**: 
    *   **`Facility`**: Physical buildings (Warehouse, Retail Store).
    *   **`FacilityLocation`**: Specific aisles, shelves, or bins within a facility.

### 6. Shipment Domain (The Logistics)
*   **Purpose**: Manages the physical movement and fulfillment of goods.
*   **Core Entities**: 
    *   **`Shipment`**: The logistics unit leaving or entering a facility.
    *   **`ShipmentItem`**: Contents of the shipment.
    *   **`PicklistItem`**: Instructions for picking items from the shelf for fulfillment.

---

## Technical Mapping & Directory Structure
Mapping of functional domains to the OFBiz `applications/` source directory:

| Domain | Directory Path | Key Contents |
| :--- | :--- | :--- |
| **Product** | `applications/product` | Product, Category, Price, Feature definitions. |
| **Order** | `applications/order` | Order processing, Requirements, Shopping Cart. |
| **Party** | `applications/party` | Person, PartyGroup, Contact Mechanisms. |
| **Inventory / Facility** | `applications/product/facility` | Inventory items, Warehousing, Shipping logic. |

---

## Secondary & Framework Domains
These domains support the core commerce engine and handle specialized or technical functions.

### 1. Accounting Domain
*   **Focus**: Financial tracking, taxes, and ledgers.
*   **Key Entities**: `Invoice`, `GeneralLedger`, `Payment`, `PaymentMethod`.

### 2. Content Domain
*   **Focus**: Digital asset management.
*   **Key Entities**: `Content`, `DataResource` (Stores images, PDFs, and documentation).

### 3. Work Effort Domain
*   **Focus**: Labor tracking and tasks.
*   **Key Entities**: `WorkEffort`, `Timesheet` (Used for tracking time spent on tasks like kitting or picking).

### 4. System & Framework Domain
*   **Focus**: Internal system management and background operations.
*   **Key Entities**: 
    *   **`UserLogin`**: Security and authentication.
    *   **`JobSandbox`**: Orchestration of background services (e.g., Shopify Sync scheduling).

---

## Summary Reference
*   **Who?** $\rightarrow$ `Party`
*   **What?** $\rightarrow$ `Product`
*   **How many?** $\rightarrow$ `Inventory`
*   **Where?** $\rightarrow$ `Facility`
*   **Promise?** $\rightarrow$ `Order`
*   **Movement?** $\rightarrow$ `Shipment`