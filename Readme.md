# Amazon Fees (GST, Referral & Closing Fee Finder)

Lightweight Node.js utilities to look up **Amazon India** GST rates, referral-fee percentages and closing-fee suggestions (by price slab). Ideal for price calculators, seller tools, dashboards, or server-side integrations.

**Exports:** `getAllCategories`, `getSubcategories`, `getGstPercent`, `getClosingFee`, `getReferralRate`
**Data required:** `amazonClosingFee.js` and `amazonData.js` (included in the repo)

---

## Table of contents

1. [Install](#install)
2. [Quick start](#quick-start)
3. [API — functions & examples](#api)

   - [`getAllCategories()`](#getallcategories)
   - [`getSubcategories(category)`](#getsubcategoriescategory)
   - [`getGstPercent(category, subcategory)`](#getgstpercentcategory-subcategory)
   - [`getClosingFee(category, subcategory, options)`](#getclosingfeecategory-subcategory-options)
   - [`getReferralRate(category, subcategory, sellingPrice, options)`](#getreferralratecategory-subcategory-sellingprice-options)

4. [Matching behaviour & notes](#matching-behaviour--notes)
5. [Errors & return shapes](#errors--return-shapes)
6. [Testing locally & publishing to npm](#testing-locally--publishing-to-npm)
7. [Contribute / TypeScript tips](#contribute--typescript-tips)
8. [FAQ](#faq)
9. [Changelog & License](#changelog--license)

---

# Install

If using locally in your project:

```bash
# If you publish to npm later:
npm install amazon-seller-fees

```

---

# Quick start

```js
const {
  getAllCategories,
  getSubcategories,
  getGstPercent,
  getClosingFee,
  getReferralRate,
} = require("amazon-seller-fees"); // or require('./path/to/index.js')

// List categories
console.log(getAllCategories());

// Get subcategories (by id or display name)
console.log(getSubcategories("automotive"));
console.log(getSubcategories("Automotive, Car & Accessories"));

// GST percent
console.log(
  getGstPercent("automotive", "Automotive - Helmets & Riding Gloves")
);

// Closing fee suggestions for price
console.log(
  getClosingFee("booksMoviesMusic", "Books", {
    price: 450,
    fulfillmentType: "selfShip",
  })
);

// Referral rate and optional closing fees
console.log(
  getReferralRate("automotive", "Automotive - Tyres & Rims", 550, {
    includeClosingFee: true,
  })
);
```

---

# API

All functions return an object with `ok: true` on success or `ok: false` with an `error` message on failure.

## `getAllCategories()`

Return list of categories:

```js
getAllCategories();
// => { ok: true, categories: [{ category_id: 'automotive', category_name: 'Automotive, Car & Accessories' }, ...] }
```

## `getSubcategories(category)`

Returns subcategories for a given category.

- `category` — string: category **id** (e.g. `'automotive'`) or **display name**.

```js
getSubcategories("automotive");
// => { ok: true, category_id: 'automotive', category_name: 'Automotive, Car & Accessories', subcategories: [{ name: 'Automotive - Helmets & Riding Gloves', gst_percent: 18 }, ...] }
```

## `getGstPercent(category, subcategory)`

Return GST percentage for a subcategory.

- `category` — string (id or display name)
- `subcategory` — string (name or fragment)

```js
getGstPercent("automotive", "Helmets & Riding Gloves");
// => { ok: true, gst_percent: 18 }
```

## `getClosingFee(category, subcategory, options = {})`

Return closing-fee slab data or computed closing fees for a given price.

**Options**

- `price` (number) — INR to pick slab
- `fulfillmentType` — `'standardEasyShip' | 'easyShipPrime' | 'selfShip' | 'sellerFlex' | 'categoriesWithException' | 'allCategories'`
- `selfShipType` — `'books' | 'allExceptBooks'` (used when `fulfillmentType === 'selfShip'`)

**Behaviors**

- If `price` present → returns fee(s) for that price (single fulfillment type or all suggestions)
- If no `price` → returns full `fixedClosingFee` slab data

**Examples**

```js
// All suggestions for INR 450
getClosingFee(null, null, { price: 450 });
// => { ok: true, fees_for_price: { standardEasyShip: 11, easyShipPrime: 11, sellerFlex: 11, selfShip: { books: 25, allExceptBooks: 25 }, ... } }

// Specific fulfillment type
getClosingFee("booksMoviesMusic", "Books", {
  price: 450,
  fulfillmentType: "selfShip",
  selfShipType: "books",
});
// => { ok:true, fulfillmentType:'selfShip', selfShipType:'books', fee:25 }
```

## `getReferralRate(category, subcategory, sellingPrice, options = {})`

Return referral percent, referral amount, GST, and optional closing fees.

**Parameters**

- `category` — string (id or name)
- `subcategory` — string
- `sellingPrice` — number (INR)
- `options`:

  - `includeClosingFee` (boolean) — attach closing fee suggestions
  - `fulfillmentType` (string) — narrow closing fee when included
  - `selfShipType` (string) — `'books' | 'allExceptBooks'` for `selfShip`

**Example**

```js
getReferralRate("automotive", "Automotive - Tyres & Rims", 550, {
  includeClosingFee: true,
});
// => { ok: true, category_id:'automotive', subcategory_name:'Automotive - Tyres & Rims', gst_percent:18, selling_price:550, referral_percent:7.0, referral_amount:38.5, closing_fee_suggestions: { ... } }
```

---

# Matching behaviour & notes

- Category input accepts **category id** (e.g. `automotive`) or **display name**.
- Subcategory matching is fuzzy: exact match → contains-match → token-score fallback.
- Price must be a non-negative number (INR). Invalid price → error.
- Closing fee dataset is read from `amazonClosingFee.fixedClosingFee`. If you want auto-selection of a default fulfillment mode, pass `fulfillmentType` or I can update the function to choose a default.

---

# Errors & return shapes

- Success: `{ ok: true, ...data }`
- Failure: `{ ok: false, error: "message" }`

Common errors:

- `Category not found`
- `Subcategory not found`
- `Invalid sellingPrice`

---

---

# License

MIT © AKHILESH SONI
