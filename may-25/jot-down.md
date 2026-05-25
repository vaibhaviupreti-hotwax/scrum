# JOT (JSON Object for Transformation)

## Main Idea
In integrations, most of the work revolves around:
- data
- data mapping
- transformations

The goal is to make transformations:
- generic
- reusable

This eventually leads to:
- faster code development
- less custom coding

❌ No need for writing too much code.

---

# Suggested Integration Approach

## Shopify → OMS

Instead of writing heavy custom logic between systems:

```text id="p8w3h1"
Shopify → JOT Layer → OMS
