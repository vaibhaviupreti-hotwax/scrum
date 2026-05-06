# Order Processing and Architecture

## Order Lifecycle
- Created
- Broker
- Allocate
- Ship
- Complete in OMS
- Shopify sync fulfillment

## Point of Sale (POS)
- Direct completed - if paid
- ATP (Available to Promise): broker
  - QOH (Quantity on Hand): completed
  - Shopify order - both decremented

## Payment Status
- Pending/Authorized

## Cart Processing
- Send sale - from shop
- Mixed cart - completed + approved

## Tool Factory Pattern
- Lifecycle with instance:
  - init()
  - getInstance()
  - destroy()
- Interface definitions
- Plugins - tool factory init to destroy lifecycle
- Cache manager / SMTP server management
- Lifecycle cleanup / instance cleanup on destroy()

## Key Components
- **hotwax-marg-util**: Authentication and 3rd party integrations
- **Order-related SQS**: Queue service for order messaging
- **mantle-shopify-connector**: Syncs and connects Shopify data via connector