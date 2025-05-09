---
title: "CSSPropertyRule: syntax property"
short-title: syntax
slug: Web/API/CSSPropertyRule/syntax
page-type: web-api-instance-property
browser-compat: api.CSSPropertyRule.syntax
---

{{APIRef("CSS Properties and Values API")}}

The read-only **`syntax`** property of the {{domxref("CSSPropertyRule")}} interface returns the literal syntax of the custom property registration represented by the {{cssxref("@property")}} rule, controlling how the property's value is parsed at computed-value time.

## Value

A string.

## Examples

This stylesheet contains a single {{cssxref("@property")}} rule. The first {{domxref("CSSRule")}} returned will be a `CSSPropertyRule` representing this rule. The `syntax` property returns the literal string `"<color>"`.

```css
@property --property-name {
  syntax: "<color>";
  inherits: false;
  initial-value: #c0ffee;
}
```

```js
const myRules = document.styleSheets[0].cssRules;
console.log(myRules[0].syntax); // "<color>"
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}
