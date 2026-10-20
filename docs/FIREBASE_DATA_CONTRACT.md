# Firebase data contract (from the supplied JSON and rules)

Source of truth is the uploaded export and rules, not the (unseen) storefront. Collection names live in `js/collections.js`.

| Collection | Fields seen in export | Admin access (rules) |
|---|---|---|
| `users/{uid}` | isActive, displayName, photoURL, phoneNumber, email, defaultAddressId, createdAt, updatedAt | get, list; update allowed by rules but not used |
| `users/{uid}/addresses/{id}` | fullName, phone, addressLine1, addressLine2, landmark, city, state, pincode, country, isDefault, createdAt, updatedAt | read |
| `products/{id}` | name, slug, description, shortDescription, seoTitle, seoDescription, categoryId, productType (`fixed`/`customizable`), basePrice, deliveryCharge, images[], primaryImage, options[], tags[], searchKeywords[], trackInventory, stockQuantity, lowStockThreshold, allowBackorder, isAvailable, isActive, isFeatured, sortOrder, reviewCount, averageRating, createdAt, updatedAt | full |
| `categories/{id}` | name, slug, description, imageURL, sortOrder, isActive, createdAt, updatedAt | full |
| `orders/{id}` | orderNumber, userId, customerName, customerEmail, customerPhone, shippingAddress{}, items[], subtotal, deliveryCharge, totalAmount, currency, paymentMethod, paymentStatus, status, customerNote, createdAt, updatedAt (+ `adminNote` allowed by rules) | read; update only status, updatedAt, adminNote on valid transitions |
| `reviews/{id}` | productId, orderId, userId, rating, reviewText, imageURLs[], isApproved, isVerifiedPurchase, createdAt, updatedAt | full |
| `wishlists/{id}` | userId, productIds[], createdAt, updatedAt | read |
| `siteSettings/public` | siteName, contactEmail, whatsappNumber, instagramURL, currency, currencySymbol, maintenanceMode, announcementEnabled, announcementText, logoURL, faviconURL, updatedAt | read, write |
| `banners/{id}` | title, subtitle, imageURL, mobileImageURL, buttonText, buttonLink, sortOrder, isActive, createdAt, updatedAt | full |
| `highlights/{id}` | title, coverImageURL, sortOrder, isActive, createdAt, updatedAt | full |
| `highlightItems/{id}` | highlightId, imageURL, caption, linkURL, sortOrder, isActive, createdAt, updatedAt | full |
| `paymentTransactions/{id}` | provider, razorpayOrderId, razorpayPaymentId, orderId, userId, amount, currency, status, signatureVerified, verifiedAt, createdAt, updatedAt | read only |
| `shipments/{id}` | orderId, userId, courierName, trackingNumber, trackingURL, status, estimatedDeliveryDate, shippedAt, deliveredAt, delayReason, createdAt, updatedAt | full |
| `customOrderRequests/{id}` | customerName, phoneNumber, contactMethod (`whatsapp`/`instagram`/`phone`), userId, productDescription, budget, requestedDate, referenceImageURLs[], status (`new` seen), adminNote, createdAt, updatedAt | read, update, delete |
| `adminActivityLogs/{id}` | action, description, targetCollection, targetDocumentId, adminEmail, adminUid, createdAt | read only |

## Known unknowns
- Shape of `products.options[]` items: the sample is empty, so Stage 2 defines the shape below for NEW records. Existing options in any other shape are preserved unchanged. The storefront must be checked against this later.
- Full set of `status` values for shipments and custom requests (only `pending` and `new` seen).
- Whether the storefront reads `logoURL`, `faviconURL`, or other fields; unverified.
- Delivery estimate (6–8 days) has no field in `siteSettings/public`; it is an app default.

## `products.options[]` shape written by this admin (new records)
```
{ "id": "size", "name": "Size", "type": "choice", "required": true,
  "choices": [ { "label": "500 g", "priceAdjustment": 0 }, { "label": "1 kg", "priceAdjustment": 300 } ] }
{ "id": "message", "name": "Message on cake", "type": "text", "required": false, "maxLength": 100, "priceAdjustment": 0 }
```
`id` is a slug of the name, unique within the product. `priceAdjustment` is in rupees and is added to `basePrice` by the storefront (unverified).
Product writes: create sets `reviewCount: 0`, `averageRating: 0`, `createdAt` and `updatedAt` (server timestamps); updates change only submitted fields plus `updatedAt`.
Category writes: `name, slug, description, imageURL, sortOrder, isActive` plus timestamps.
