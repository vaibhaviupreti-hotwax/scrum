# searchFormMap vs EntityFind: Feature Gap in `s1`/`e1` REST Transitions

## Purpose

Moqui's `EntityFind` interface supports several query-shaping capabilities that
are not reachable through the `searchFormMap`/`searchFormInputs` mechanism used
by the generic REST entity/service transitions (`s1`, `e1` in
`runtime/base-component/webroot/screen/webroot/rest.xml`). This means a caller
hitting `GET /rest/e1/...` or `GET /rest/s1/...` cannot request these features
via query/form parameters — they can only be applied by writing custom
Groovy/service code that calls the `EntityFind` API directly.

## Background

- `EntityFind.searchFormMap(inputFieldsMap, defaultParameters, skipFields, defaultOrderBy, alwaysPaginate)`
  is implemented in
  `framework/src/main/groovy/org/moqui/impl/entity/EntityFindBase.groovy:380-428`,
  with the field-condition logic in `processInputFields`
  (`EntityFindBase.groovy:430-542`).
- The generic entity/service REST transitions
  (`runtime/base-component/webroot/screen/webroot/rest.xml`, `e1` at line 228,
  `s1` at line 358) delegate to `ec.web.handleEntityRestCall(...)` /
  `ec.web.handleServiceRestCall(...)`, which for entity list GETs routes
  through `framework/src/main/groovy/org/moqui/impl/service/RestApi.groovy:369,377,397`:
  ```groovy
  ec.entity.find(entityName).searchFormMap(ec.context, null, null, null, false)
  ```
- From `inputFieldsMap`/query params, `searchFormMap` only wires up: field
  equality/range conditions, `orderByField`, `pageIndex`/`pageSize`/`pageNoLimit`
  (offset/limit), and forces `useCache(false)`. Nothing else on the
  `EntityFind` object is touched.

## Gaps

### 1. `distinct`

- `EntityFind.distinct(boolean)` — `EntityFind.java:172`, impl
  `EntityFindBase.groovy:606-607`.
- No `inputFieldsMap` key sets `this.distinct`. A caller cannot request
  `DISTINCT` results through the generic REST entity/service endpoints; it
  must be set in code before/after `searchFormMap` runs.

### 2. `fieldsToSelect` (`selectField`/`selectFields`)

- `EntityFind.selectField(String)` / `selectFields(Collection)` —
  `EntityFind.java:130,135`, impl `EntityFindBase.groovy:546-566`.
- No query-string parameter populates `fieldsToSelect`. Every generic REST
  list call returns the full field set for the entity/view-entity; there is no
  way to ask for a projection (e.g. `?fieldsToSelect=orderId,statusId`)
  through `s1`/`e1`.

### 3. `filterByDate` (effective-date / "as of" filtering)

- `EntityConditionFactory.makeConditionDate(fromField, thruField, moment)` —
  `EntityConditionFactory.java:50` — and `EntityList.filterByDate(fromDateName,
  thruDateName, moment[, ignoreIfEmpty])` — `EntityList.java:47-48`.
- These compare **two fields** (`fromDate`/`thruDate`) against a single
  "as of" instant — the standard Moqui/OFBiz effective-dating pattern used for
  things like `StatusValidChange`, price/promo effective windows, etc.
- `processInputFields` (`EntityFindBase.groovy:506-538`) only supports
  independent single-field range params: `<field>_from`, `<field>_thru`,
  `<field>_period`/`_poffset`/`_pdate`. That is a different mechanism and
  cannot express "moment falls between fromDate and thruDate" as a single
  condition. There is no `filterByDate` equivalent exposed via query params.

## Net effect

Any REST consumer of the generic `s1`/`e1` entity/service list transitions is
limited to filtering (equality + from/thru ranges), ordering, and pagination.
`distinct`, column projection (`fieldsToSelect`), and two-field effective-date
filtering (`filterByDate`) all require a purpose-built service or screen
transition that calls `EntityFind` directly instead of relying on
`searchFormMap`.
