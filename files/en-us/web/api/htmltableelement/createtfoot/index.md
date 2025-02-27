---
title: "HTMLTableElement: createTFoot() method"
short-title: createTFoot()
slug: Web/API/HTMLTableElement/createTFoot
page-type: web-api-instance-method
browser-compat: api.HTMLTableElement.createTFoot
---

{{APIRef("HTML DOM")}}

The **`createTFoot()`** method of
{{domxref("HTMLTableElement")}} objects returns the {{HTMLElement("tfoot")}} element
associated with a given {{HtmlElement("table")}}. If no footer exists in the table, this
method creates it, and then returns it.

> [!NOTE]
> If no footer exists, `createTFoot()` inserts a new
> footer directly into the table. The footer does not need to be added separately as
> would be the case if {{domxref("Document.createElement()")}} had been used to create
> the new `<tfoot>` element.

## Syntax

```js-nolint
createTFoot()
```

### Parameters

None.

### Return value

{{domxref("HTMLTableSectionElement")}}

## Examples

```js
let myFoot = myTable.createTFoot();
// Now this should be true: myFoot === myTable.tFoot
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}
