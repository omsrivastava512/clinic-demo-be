# How commerce platforms model catalogs and billing

Oct 21, 2026 · @Om Srivastava

## Summary

The same few designs repeat across the platforms I could read: a shared platform under separate app shells, a typed core product record, variants as SKU-level children of a parent, combos as a priced record plus a recipe expanded at pick time, one base unit with conversion factors, and invoice lines that copy what was sold. Most of Swiggy, Zomato, Blinkit, Amazon and Flipkart is not public, so for those I say so and name the closest documented analogue.

**How to read the tags.** \[Confirmed\] means stated in a primary source I opened (a company statement, official docs, or code at a pinned commit) and linked. \[Inferred\] means my reasoning from confirmed facts, with the reasoning stated; it also covers anything that rests only on press coverage. \[Unknown\] means I found no public evidence.

**Corrections to the brief.** Bolt launched as a tile on the Food page, not a tab. Toing is a standalone app positioned on affordability, not daily deals. Instamart also has its own app, in addition to its place in the main app. Instamart's 10 to 15 minute promise is not confirmed by any primary source I found.

**Key takeaways**

- **Task 1a, shared or separate:** Swiggy ships several apps over shared capabilities. Toing is a separate app that reuses Swiggy's tech and its restaurant and delivery infrastructure, and Bolt orders are indistinguishable from Food orders to delivery partners. Whether any of it shares a database or login is \[Unknown\]. Grab, Uber and DoorDash document one shared order or fulfillment platform that new verticals extend.
- **Task 1b, one table or many:** none of the platforms I could read uses one flat table or a table per kind. The common shape is a core product with a type column, attributes in EAV or JSON, and side tables for bundles, downloads and gift cards. Amazon's public API shows one generic item plus a per-type JSON schema. Restaurant menus add modifier groups and time-based availability; grocery catalogs add store-level price and stock.
- **Task 2a, variants:** the sellable SKU is always its own record tied to a parent. Shopify, Saleor, Medusa, Spree and Odoo use a child table; Magento, Amazon, eBay and Walmart keep each variant as a full item grouped by a link or key.
- **Task 2b, combos:** a combo is its own priced sellable plus a recipe of components that is multiplied out (order quantity × component quantity) into stock moves and picks. Odoo's kit explosion and Medusa's inventory kit are the clearest documented mechanics.
- **Task 2c, units:** one base unit per item with a factor to every other unit (Odoo has a real table; Magento and Medusa keep a flag or free text). Weighted items keep ordered, fulfilled and charged quantities apart, and Instacart documents three modes. Standards: UN/CEFACT Recommendation 20 codes and GS1 `netContent`.
- **Task 3, billing:** the invoice or order line is its own row that copies quantity, price and tax at the time of sale, and the per-type rule (quantity × rate, or plan × duration) runs once. Subscriptions sit in a separate billing layer (Stripe, Shopify selling plans, Medusa's subscription module). A worked example with numbers is in Task 3.

## Product facts check

Bolt, Toing and Instamart all exist, but the brief's picture needs three corrections: Bolt was announced as a tile on the Food page rather than a third tab, Instamart is also a standalone app, and Toing is pitched on low prices rather than rotating daily deals. Checked on 22 and 23 Oct 2026. Company statements are tagged \[Confirmed\]; anything resting only on press reports is tagged \[Inferred\] and says so.

| Brief says | What I found | Tag | Source |
| --- | --- | --- | --- |
| The name is “Bolt” | Swiggy calls its quick-delivery food offering “Bolt”. At launch it meant quick-to-prepare meals from nearby restaurants and quick-service chains, delivered in about 10 minutes within roughly 2 km, in 6 cities. | \[Confirmed\] | [Swiggy press release, Oct 2024](https://www.swiggy.com/corporate/press-release/swiggy-launches-bolt-delivering-your-favorite-food-in-just-10-minutes/) |
| Bolt is a tab next to Food and Instamart | The launch release says a “Bolt – Food in 10 mins” tile would be featured on the Food page of the Swiggy app. The May 2025 expansion release again calls it a tile on the Food section, and the Jul 2026 letter calls it “our faster Food delivery proposition”. I found no source for where it sits in the app today (it may have moved). | \[Confirmed\] for the 2024 to 2026 statements; \[Unknown\] for today | [May 2025 release](https://nsearchives.nseindia.com/corporate/SWIGGY_02052025162900_CoveringLetter_Press_Release_Bolt_Expansion.pdf), [Jul 2026 letter](https://nsearchives.nseindia.com/corporate/SWIGGY_30072026154100_Covering_Letter_Letter_to_Shareholders_-_signed.pdf) |
| (not in the brief) Bolt's footprint | 6 cities at launch, 500+ cities in the May 2025 release, 700+ cities in the Jul 2026 letter. | \[Confirmed\] | The releases and letter above |
| Instamart is a tab in the main app | It is, and a standalone Instamart app also exists, published by Swiggy Instamart Private Limited. Business Today dates the standalone launch to Jan 2025 and quotes Swiggy's CEO saying Instamart remains inside the Swiggy app. | \[Confirmed\] that the standalone app exists; \[Inferred\] for the launch date and the CEO quote (press only) | [App Store listing](https://apps.apple.com/in/app/instamart-10-mins-grocery-app/id6738619733), [Business Today, 13 Jan 2025](https://www.businesstoday.in/amp/technology/news/story/swiggy-launches-instamart-as-a-standalone-app-available-on-android-and-ios-460547-2025-01-13) |
| Instamart delivers in roughly 10 to 15 minutes | Not confirmed. The current App Store listing states no delivery time. | \[Unknown\] | [App Store listing](https://apps.apple.com/in/app/instamart-10-mins-grocery-app/id6738619733) |
| Toing is a standalone Swiggy app | Yes. The same letter calls it a standalone app for affordable meals, live in about 50 cities, run as a separate app so Swiggy can test the affordability model. Swiggy's food marketplace CEO called it a clean experiment on the marketplace model. | \[Confirmed\] | Shareholder letter above; [Business Today interview, 31 Dec 2025](https://www.businesstoday.in/amp/magazine/interview/story/trying-multiple-ways-to-succeed-swiggy-food-marketplace-ceo-rohit-kapoor-508841-2025-12-31) |
| Toing is for aggressive daily restaurant deals | The sources describe affordability and lowest price, not daily deals. A Sep 2025 press report says the app was serving select locations in Pune, claimed meals under Rs 99 and promised free delivery above Rs 99. | \[Confirmed\] for affordability; \[Inferred\] for Pune and the Rs 99 figures (press only); “daily deals” is unsupported | [Storyboard18, 15 Sep 2025](https://www.storyboard18.com/brand-makers/-80944.htm) |
| Toing shares code, database and identity with the main app | The letter says Toing launched by leveraging existing tech and execution capabilities, with restaurant and delivery infrastructure already in place. Nothing I found says whether it shares logins, user accounts or databases. | \[Confirmed\] that it reuses existing capabilities; \[Unknown\] for login, accounts and database | Shareholder letter |

One more hint on the app question: Swiggy's engineering blog says the company runs several business lines and shell apps on Android and iOS with tooling shared across all of them, but it names none of the apps ([Swiggy Bytes, Jul 2026](https://medium.com/swiggy-bytes/how-we-made-all-swiggy-apps-operations-ai-native-1343612ec228)). \[Confirmed\] as a statement; it says nothing about backends or databases.

## Task 1a: Shared versus separate backends

Swiggy's own statements show several consumer apps sitting on shared capabilities, but nothing public says whether Food, Instamart, Bolt and Toing share databases \[Unknown\]. The documented analogues (Grab, Uber, DoorDash) all describe one shared order or fulfillment platform that each new vertical extends, with the vertical-specific parts at the edges.

### Swiggy: what the sources say

| Surface | What the sources say | Tag and source |
| --- | --- | --- |
| Bolt | Featured as a tile on the Food page at launch (Oct 2024) and again as a tile on the Food section in the May 2025 expansion release, which describes a “curated menu of quick-serve, high-demand items that have minimal or no preparation time”. The Jul 2026 letter calls it “our faster Food delivery proposition”. Delivery partners “are not informed of the distinction between Bolt and regular orders” (2024) and “are not informed that an order is a Bolt one” (2025). | \[Confirmed\] [Oct 2024 release](https://www.swiggy.com/corporate/press-release/swiggy-launches-bolt-delivering-your-favorite-food-in-just-10-minutes/), [May 2025 release](https://nsearchives.nseindia.com/corporate/SWIGGY_02052025162900_CoveringLetter_Press_Release_Bolt_Expansion.pdf), [Jul 2026 letter](https://nsearchives.nseindia.com/corporate/SWIGGY_30072026154100_Covering_Letter_Letter_to_Shareholders_-_signed.pdf) |
| Instamart | A standalone Instamart app exists, published by Swiggy Instamart Private Limited. Business Today reported its Jan 2025 launch and quoted Swiggy's CEO saying the standalone app “will be an add-on” and that Instamart “will remain a key offering within the Swiggy app”. | \[Confirmed\] that the app exists ([App Store listing](https://apps.apple.com/in/app/instamart-10-mins-grocery-app/id6738619733)); \[Inferred\] for the launch date and the CEO's remarks (press only: [Business Today](https://www.businesstoday.in/amp/technology/news/story/swiggy-launches-instamart-as-a-standalone-app-available-on-android-and-ios-460547-2025-01-13)) |
| Toing | A standalone app that “runs on a separate app”, called a “separate platform” in another answer, housed in “Platform Innovations”, which the letter calls a “deliberate sandbox”. Launched “without any significant incremental fixed costs” by “leveraging our existing tech and execution capabilities”, with the “restaurant/delivery infrastructure” already built. | \[Confirmed\] [Jul 2026 letter](https://nsearchives.nseindia.com/corporate/SWIGGY_30072026154100_Covering_Letter_Letter_to_Shareholders_-_signed.pdf) |
| Other apps | Business Today calls Snacc a separate Swiggy app that delivers food in 15 minutes. Swiggy Bytes says the “Swiggy Apps” ecosystem spans multiple business lines and shell apps on Android and iOS, with tooling shared across them. | \[Inferred\] for Snacc (press only); \[Confirmed\] for the Swiggy Bytes statement ([post](https://medium.com/swiggy-bytes/how-we-made-all-swiggy-apps-operations-ai-native-1343612ec228)) |

What that suggests about the backend:

- **Bolt shares the Food pipeline.** Couriers cannot tell a Bolt order from a regular one, which points to one order and dispatch flow \[Inferred\]. Whether Bolt listings are rows in the Food catalog, a copy, or a separate catalog is \[Unknown\]; see below.
- **A vertical can sit behind more than one app shell.** Instamart is both a tab in the main app and its own app, so its backend has to serve two clients \[Inferred\]. Whether it shares order, catalog or user stores with Food is \[Unknown\].
- **Toing reuses supply and tech.** Swiggy says so directly \[Confirmed\]. The letter's metrics (two in three new Toing users are new to the platform or dormant; some Toing restaurants do more than half their Swiggy-platform volume there) imply Toing users and restaurants are matched to Swiggy's existing identities, at least for analytics \[Inferred: my reading of the letter\]. Whether Toing shares logins, accounts or databases is \[Unknown\].

### How one app can present several verticals

No Swiggy source describes this. The patterns in the brief, and what is documented elsewhere:

| Pattern | Public evidence | Tag and source |
| --- | --- | --- |
| API gateway or edge layer | Uber groups services into domains and domains into five layers (infrastructure, business, product, presentation, edge); layers depend only on the layers below, and a gateway is “a single entry-point into a collection of underlying services” | \[Confirmed\] [Uber, domain-oriented architecture](https://www.uber.com/us/en/blog/microservice-architecture/) |
| Modular client in one app | Grab's app has feature modules that do not depend on each other, kit modules for communication between them, core modules (networking, analytics, experimentation, storage, config), shared library modules, and an app module that wires them together with dependency injection; how Food or Mart plug in is not described | \[Confirmed\] [Grab, app modularisation](https://engineering.grab.com/app-modularisation-at-scale) |
| Backend-for-frontend, micro-frontends | no public description found at any company I checked | \[Unknown\] |
| Service per vertical | not documented for Swiggy; the companies below describe a shared order or fulfillment platform instead | \[Unknown\] for Swiggy |

### Analogues: how a new vertical attaches to a shared platform

| Company | Shared piece | How the new vertical attaches | Tag and source |
| --- | --- | --- | --- |
| Grab | One Order Platform processes GrabFood and GrabMart orders; DynamoDB serves the OLTP queries and MySQL RDS the OLAP queries | GrabMart was built by “leveraging the GrabFood platform”: it began as a category within GrabFood, then the order management system was changed “to account for the new mart order type” and allocation was changed to assign the right type of driver; browsing became item-first rather than merchant-first | \[Confirmed\] [order platform](https://engineering.grab.com/how-we-store-millions-orders), [GrabMart](https://engineering.grab.com/grabmart-product-team-experience) |
| Uber | One fulfillment platform that, per its engineers, powers physical fulfillment for all Uber verticals, built on fulfillment entities (for example an order entity and a job entity), each defined statically through a statechart configuration, with Google Cloud Spanner as storage; every Uber product and city was migrated to it | a vertical is configuration plus product-layer services on the shared platform | \[Confirmed\] for the platform and storage; \[Inferred\] for the last column ([Uber](https://www.uber.com/us/en/blog/fulfillment-platform-rearchitecture/)) |
| DoorDash | For grocery and convenience (“new verticals” in its wording) a facade service, Retail Fulfillment Data Service, fronts a `store_items` table holding catalog, inventory and pricing data; storage moved from PostgreSQL to CockroachDB | whether restaurants use the same service or database is not stated | \[Confirmed\] [DoorDash](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/); \[Unknown\] for restaurants |
| Gojek | All driver partners had GoPay e-wallets, and the GoResto merchant app gave GoFood merchants the same kind of wallet; GoResto evolved into GoBiz, which the post calls “merchant SuperApp”; Gojek had deprecated its legacy codebase in favour of microservices | whether Food and Mart share services is not stated | \[Confirmed\] [Gojek](https://blog.gojek.io/the-road-to-a-merchant-superapp) |

### Bolt and the Food catalog

No source says how Bolt's listings are stored. The evidence narrows the options: Bolt items come from restaurant chains already on Swiggy (the releases name KFC, McDonald's and Burger King, among others), the menu is described as curated, and delivery partners cannot tell a Bolt order from a regular one. The simplest design consistent with that is a filtered view of the Food catalog, a flag on existing menu items plus rules on distance and preparation time \[Inferred\]. A curated copy or a separate catalog would also fit the statements \[Unknown\].

```sql
-- Illustration of the filtered-view option, not Swiggy's schema
restaurant (restaurant_id PK, name, lat, lng)
menu_item  (item_id PK, restaurant_id FK, name, price, prep_time_sec, is_bolt_eligible BOOLEAN)

-- a Bolt listing is the Food listing narrowed by rules
SELECT i.* FROM menu_item i JOIN restaurant r USING (restaurant_id)
WHERE i.is_bolt_eligible AND i.prep_time_sec <= :max_prep AND distance_km(r, :customer) <= 2;
```

### Toing: code, database and identity

| Question in the brief | Answer | Tag |
| --- | --- | --- |
| Separate codebase and database from the main app? | It is a separate app; whether it is a separate repository or database is not stated | \[Confirmed\] for the app; \[Unknown\] for the rest |
| Does it reuse restaurant, menu and order systems? | Swiggy says it reuses existing tech and its restaurant and delivery infrastructure; menu and order systems are not named | \[Confirmed\] for the statement; \[Unknown\] for the specifics |
| One account and identity across both? | Not stated; the new-user metrics suggest identities are matched to the Swiggy platform | \[Unknown\]; \[Inferred\] for the matching |

### Zomato and Blinkit

Eternal's Q4 FY25 letter says about a third of going-out GOV went through the District app and that the going-out offering will be accessible only on the District app, so the group moves product lines into separate apps \[Confirmed: [Eternal letter](https://b.zmtcdn.com/investor-relations/d9c290cd23764a09789769c39682276a_1746094084.pdf)\]. Business Today refers to the Blinkit app \[Inferred: press only\], and Wikipedia says the Zomato app kept its name after the Eternal rename \[Inferred: secondary source\]. I found no primary engineering source on Zomato's or Blinkit's databases, catalogs or service boundaries \[Unknown\]. The only items I found were third-party (a MongoDB case study, vendor blog posts, a job listing) and I did not rely on them. For the same question about a quick-commerce vertical beside a restaurant business, the closest documented analogues are Grab's GrabMart built on GrabFood and DoorDash's retail path above.

## Task 1b: One table or many for things you can buy

No platform I could read uses one flat table for everything, and none splits the core catalog into a table per kind; the common shape is a core product record with a type, type-specific attributes held in EAV or JSON, and separate tables only for behavior that really differs (bundles, downloads, gift cards, subscriptions).

### The patterns and who is documented using them

| Pattern | What it looks like | Documented examples | Tag and source |
| --- | --- | --- | --- |
| Single table with a type column | one product row whose `type` decides which rules apply | Odoo: `product.template.type` is goods (`consu`), `service` or `combo`. Magento: `catalog_product_entity.type_id`, with types such as simple, bundle, virtual and downloadable. Saleor: `ProductType.kind` is normal or gift card | \[Confirmed\] [Odoo](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_template.py#L63), [Magento table](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Catalog/etc/db_schema.xml#L10), [Magento types](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Catalog/Model/Product/Type.php), [Saleor](https://github.com/saleor/saleor/blob/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/product/models.py#L134) |
| Base table plus extension tables | the core row stays generic and type-specific data sits in side tables | Magento: bundle options and selections, `downloadable_link`, configurable super-attribute and super-link tables. Odoo: `product.combo` and `product.combo.item`, and `mrp.bom` for kits | \[Confirmed\] [Magento bundle](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Bundle/etc/db_schema.xml#L10), [Odoo combo](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_combo.py) |
| EAV (entity, attribute, value) | attributes are rows, not columns | Magento: `eav_attribute` plus value tables such as `catalog_product_entity_varchar` and `catalog_product_entity_decimal` (`value_id`, `attribute_id`, `store_id`, `entity_id`, `value`). Saleor: attributes are attached to a product type and values are stored in assignment tables (`AttributeProduct`, `AttributeVariant`, `AssignedProductAttributeValue`, `AssignedVariantAttributeValue`) | \[Confirmed\] [Magento EAV](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Eav/etc/db_schema.xml#L274), [Saleor attribute models](https://github.com/saleor/saleor/tree/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/attribute/models); calling Saleor's design EAV-like is my label \[Inferred\] |
| JSON or metadata columns | free-form data in a JSON column | Medusa: `metadata` on inventory items and `variant_option_values` as json on order lines. Saleor: `ModelWithMetadata` on product types, products and order lines. DoorDash: `store_items` columns grouped into JSONB columns | \[Confirmed\] [Medusa inventory item](https://github.com/medusajs/medusa/blob/f7add009dd2f142d98090a5194750e71b2147ddf/packages/modules/inventory/src/models/inventory-item.ts), [Medusa line item](https://github.com/medusajs/medusa/blob/f7add009dd2f142d98090a5194750e71b2147ddf/packages/modules/order/src/models/line-item.ts), [Saleor](https://github.com/saleor/saleor/blob/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/order/models.py#L567), [DoorDash](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/) |
| Separate model for behavior that differs | its own tables, linked from the order line | Saleor has a `GiftCard` model; Spree has `Spree::GiftCard` and a digital-asset model for downloads | \[Confirmed\] [Saleor](https://github.com/saleor/saleor/blob/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/giftcard/models.py), [Spree models](https://github.com/spree/spree/tree/a0e5b66419569d1e91d3b1eaf180444b73c65cd1/spree/core/app/models/spree) |

I found no platform using a full table per product kind for the core catalog \[Inferred: from the code I read; Amazon's internal storage is unknown\].

### Amazon

Amazon's public catalog API returns one `Item` shape for every product, with the fields `asin`, `attributes`, `classifications`, `dimensions`, `identifiers`, `images`, `productTypes`, `relationships`, `salesRanks`, `summaries` and `vendorDetails` \[Confirmed: [Catalog Items reference](https://developer-docs.amazon/sp-api/docs/catalog-items-api-v2022-04-01-reference)\]. A separate Product Type Definitions API serves the attribute and data requirements for items in the catalog as JSON schemas that extend JSON Schema 2019-09 \[Confirmed: [Product Type Definitions](https://developer-docs.amazon/sp-api/docs/product-type-definitions-api)\]. So the public contract is one generic item record plus a per-product-type schema for its `attributes` \[Inferred: a description of the API, not of Amazon's storage\]. Amazon's bundling guidance lists gift cards, digital items and subscription products as ineligible for virtual bundles \[Confirmed: [Amazon selling blog](https://sell.amazon.com/blog/product-bundling)\], which suggests they are catalog products excluded by a rule rather than things held in a different system \[Inferred\]. How Amazon stores any of this internally, and how Prime plans are modeled, is \[Unknown\]. The closest documented analogues are Magento's EAV with `type_id` and Saleor's product types.

### Blinkit: an ambulance beside groceries

What is public is thin, and all of it is press coverage of a Jan 2025 launch in Gurugram; I found no primary Blinkit page \[Inferred: press only\]. [Business Today](https://www.businesstoday.in/amp/india/story/profit-is-not-a-goal-here-blinkit-rolls-out-10-minute-ambulance-service-starting-in-gurugram-459361-2025-01-02), [MediaNama](https://www.medianama.com/2025/01/223-demand-and-scalability-questions-around-blinkits-10-min-ambulance-service/) and [YourStory](https://yourstory.com/2025/01/blinkit-launches-ten-minute-ambulance-service-gurugram) agree that users can book a Basic Life Support (BLS) ambulance through the Blinkit app, staffed by a paramedic, an assistant and a driver; Business Today quotes Blinkit's CEO saying profit is not a goal. Two details come from one outlet each: YourStory says the option appears in the app only once the service reaches a customer's area, and that it was in a testing phase in selected parts of Gurugram; MediaNama says the price starts at Rs 2,000, but from a sample image Blinkit posted on X rather than an official price list. Neither MediaNama nor YourStory says who operates the ambulances. Whether the booking goes through the cart, how payment works and how the service is modeled in the catalog are \[Unknown\]. Two designs for a service beside goods are documented elsewhere:

- **A catalog item of a service type** that skips shipping and stock: Odoo's `service` type, a Saleor product type with shipping turned off, Shopify's `requiresShipping` flag, Magento's `virtual` type \[Confirmed: fields in the code and docs linked above\].
- **A separate booking domain** that reuses only checkout and payment: the layered split in Uber's DOMA post, where product-layer services sit on shared business and infrastructure layers \[Inferred: my mapping\].

Which one Blinkit uses is \[Unknown\].

### Restaurant menus versus grocery catalogs

The structures differ because a restaurant item is customized and a grocery item is a stocked product.

| Dimension | Restaurant menu (Uber Eats Menu API) | Grocery catalog (Instacart, DoorDash) |
| --- | --- | --- |
| Shape | menus group categories, categories group items; an item can be a parent item or a modifier option | a product carries name, description, photos, nutrition and size or weight, plus store-specific attributes such as availability and location-by-location pricing |
| Add-ons | modifier groups attach to an item, with `min_permitted` and `max_permitted` quantities; modifier options are items themselves and the same item can sit in several groups; an item's price can be overridden in the context of a modifier group | no add-on concept in what I read; shoppers suggest replacement items from the catalog when something is out of stock |
| Availability | menus carry `service_availability` by day and time; items carry `visibility_info` | availability is a store-level attribute; DoorDash's `store_items` holds catalog, inventory and pricing together |
| Identity | merchant-defined ids with `external_data` | UPC and SKU (Instacart); each SKU is a list of product attributes with a brand taxonomy (DoorDash) |

&#91;Confirmed\] for both columns. Restaurant side: [Uber Eats menu guide](https://developer.uber.com/docs/eats/guides/menu-integration) and [example menu payloads](https://developer.uber.com/docs/eats/references/api/v2/example-menu-payloads). Grocery side: [Instacart catalog story](https://company.instacart.com/tech-innovation/the-story-behind-an-instacart-order-part-1-building-a-digital-catalog), [Instacart catalog docs](https://docs.instacart.com/connect/fulfillment_guide/concepts/catalog/), [DoorDash retail fulfillment](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/) and [DoorDash knowledge graph](https://careersatdoordash.com/blog/building-doordashs-product-knowledge-graph-with-large-language-models/). The “add a Coke to your burger” case maps to a modifier option in a modifier group on the burger \[Inferred: my mapping onto the Uber Eats structure\].

### Simplified schemas

Illustrations of the patterns, not any company's schema:

```sql
-- 1. single table with a type column
product (product_id PK, type CHECK (type IN ('GOODS','SERVICE','COMBO','GIFT_CARD')),
         name, price, requires_shipping BOOLEAN, track_stock BOOLEAN, attributes JSON)

-- 2. base table plus extension tables
product          (product_id PK, type, name, price)
bundle_component (bundle_id FK, component_id FK, qty)
digital_asset    (product_id FK, file_url, max_downloads)

-- 3. EAV
attribute       (attribute_id PK, code, value_type)
attribute_value (product_id FK, attribute_id FK, value_text, value_number, PRIMARY KEY (product_id, attribute_id))
```

The restaurant-menu shape, simplified from the Uber Eats structure:

```sql
menu                (menu_id PK, store_id FK)
category            (category_id PK, menu_id FK, title)
item                (item_id PK, store_id FK, title, price)
category_item       (category_id FK, item_id FK)
modifier_group      (group_id PK, title, min_permitted, max_permitted)
item_modifier_group (item_id FK, group_id FK)
modifier_option     (group_id FK, item_id FK, price_override)     -- the option is itself an item
```

### Which pattern to pick

My reading \[Inferred\], not a documented rule: a core `product` table with a type, schema-validated attributes in JSON or EAV, and side tables only where fulfillment or billing behaves differently (bundles, downloads, gift cards, subscriptions). That is where Odoo, Magento and Saleor converge.

## Task 2a: Variants

Every platform I could read keeps the sellable unit (own barcode, price and stock) as its own record and ties several of them to a parent that holds the shared content. The difference is storage: Shopify, Saleor, Medusa, Spree and Odoo put parent and variant in different tables, while Magento's configurable products, Amazon's parent and child ASINs, eBay and Walmart keep every variant as a full item and group them with a key or a relationship. The shopper sees one page either way; the storage choice decides whether a variant can also exist on its own.

| Platform | Parent → variant | How they are tied | Tag and source |
| --- | --- | --- | --- |
| Shopify | product → variant | variants belong to a product; the limit is 2,048 variants per product | \[Confirmed\] [Shopify developer community announcement](https://community.shopify.dev/t/the-product-variant-limit-is-now-2048-for-all-merchants/23938) |
| Saleor | `Product` → `ProductVariant` | `ProductVariant.product` foreign key and a unique `sku`; `Product.default_variant` is a one-to-one pointer to the default variant; variant attributes are declared per product type (`AttributeVariant`) and stored as `AssignedVariantAttributeValue` rows | \[Confirmed\] [Product](https://github.com/saleor/saleor/blob/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/product/models.py#L183), [ProductVariant](https://github.com/saleor/saleor/blob/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/product/models.py#L362), [attribute models](https://github.com/saleor/saleor/tree/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/attribute/models) |
| Medusa v2 | `Product` → `ProductVariant` | `product` relation; option values attach through the pivot `product_variant_option` | \[Confirmed\] [product-variant.ts](https://github.com/medusajs/medusa/blob/f7add009dd2f142d98090a5194750e71b2147ddf/packages/modules/product/src/models/product-variant.ts) |
| Spree | `Product` → `Variant` | `option_values` through `option_value_variants`; the product has a `default_variant`, called the “face” of the product in the code | \[Confirmed\] [product.rb](https://github.com/spree/spree/blob/a0e5b66419569d1e91d3b1eaf180444b73c65cd1/spree/core/app/models/spree/product.rb), [variant.rb](https://github.com/spree/spree/blob/a0e5b66419569d1e91d3b1eaf180444b73c65cd1/spree/core/app/models/spree/variant.rb) |
| Odoo 19 | `product.template` → `product.product` | `product_tmpl_id`; the variant stores `product_template_attribute_value_ids` and a `combination_indices` string; the template lists its axes in `attribute_line_ids` | \[Confirmed\] [product\_product.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_product.py), [product\_template.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_template.py) |
| Magento 2 | configurable product row → child product rows | `catalog_product_super_attribute(product_id, attribute_id, position)` names the varying attributes; `catalog_product_super_link(product_id, parent_id)` joins each child to its parent, and both columns are foreign keys into `catalog_product_entity`, so a child is an ordinary product row | \[Confirmed\] [db\_schema.xml](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/ConfigurableProduct/etc/db_schema.xml#L60) |
| Amazon | parent ASIN → child ASINs | relationship attributes are set on each child; the catalog API returns `relationships` per marketplace, each with a `type` (for example VARIATION), `parentAsins`, `childAsins` and a `variationTheme` such as COLOR\_NAME/STYLE\_NAME; size, color and style are the core themes that remain after Sep 2025; the Product Type Definitions page refers to `parentageLevel` values CHILD, PARENT and NONE | \[Confirmed\] [Catalog Items reference](https://developer-docs.amazon/sp-api/docs/catalog-items-api-v2022-04-01-reference), [SP-API changelog](https://developer-docs.amazon/sp-api/changelog/upcoming-removal-of-unused-variation-themes-in-the-listings-items-api), [Product Type Definitions](https://developer-docs.amazon/sp-api/docs/product-type-definitions-api) |
| eBay | inventory item group → inventory items | the group has a key, title, description and `variesBy.specifications`; members are listed in `variantSKUs` and must already exist as inventory items | \[Confirmed\] [eBay docs](https://developer.ebay.com/api-docs/sell/static/inventory/inventory-item-groups.html) |
| Walmart | variant group → items | every item in a group shares a `variantGroupId` and `variantAttributeNames`; exactly one item is the `isPrimaryVariant`; up to three variant attributes; an item can belong to only one group | \[Confirmed\] [Walmart docs](https://developer.walmart.com/us-marketplace/docs/setting-up-multiple-variants-of-an-item) |
| Etsy | listing → products | inside a listing's inventory each product is one combination of property values with its own `sku` and `offerings` (price and quantity) | \[Confirmed\] [Etsy docs](https://developers.etsy.com/documentation/tutorials/listings/) |

### Separate listings or child records?

Both exist.

- **Child rows in another table** (Shopify, Saleor, Medusa, Spree, Odoo): a variant cannot exist without its parent, and shared content such as title, description and brand is stored once. \[Confirmed: table above\]
- **Full items joined by a link** (Magento, Amazon, eBay, Walmart): each child already has its own identity, so it can be created and sold before or without a parent. Amazon's changelog says children left unchanged when a parent is deleted become standalone listings that stay buyable \[Confirmed\]. I read that as the parent being a grouping layer and the child a complete listing \[Inferred\].
- **Marketplaces lean to the second shape.** Three of the four marketplaces I read (Amazon, eBay, Walmart) use it, and Etsy stores variants inside the listing \[Confirmed\]. My reading is that outside sellers already identify each variant by its own SKU or barcode, so grouping afterwards is easier than forcing a parent first \[Inferred\].

### A simplified schema

```sql
product              (product_id PK, title, description, brand_id)                  -- shared content
option               (option_id PK, product_id FK, name)                            -- Size, Flavor
option_value         (option_value_id PK, option_id FK, value)                      -- M, Masala
variant              (variant_id PK, product_id FK, sku UNIQUE, barcode, price, weight)   -- the sellable unit
variant_option_value (variant_id FK, option_value_id FK, PRIMARY KEY (variant_id, option_value_id))
stock                (variant_id FK, warehouse_id FK, on_hand, reserved, PRIMARY KEY (variant_id, warehouse_id))
```

This is simplified from Medusa (the `product_variant_option` pivot) and Spree (`option_value_variants`); Odoo and Saleor use the same join idea under other names. The brief's SPU/SKU split maps onto it as product = SPU and variant = SKU. That mapping is mine \[Inferred\]: SPU/SKU is marketplace shorthand, and the platforms above say product/variant, template/product or parent/child.

### Chips flavors, shirt sizes, phone storage

A shirt in S/M/L and a phone in 128/256 GB are the textbook case: one product, one or two option axes, one variant per combination, each with its own price and stock \[Inferred: how the schema above is normally used\]. Chips flavors are a catalog decision rather than a schema one: flavors can be options of one product (one page with a flavor picker) or separate products with their own barcodes and pages, grouped by a link if at all \[Inferred\]. Which choice Swiggy Instamart or Blinkit made is \[Unknown\]; I found no public catalog documentation for either.

Flipkart's own variant documentation: I found only third-party integration guides (for example [Fynd](https://documentation.fynd.com/commerce/docs/openapi/listings-management/flipkart/faq-flipkart) and [Streamoid](https://streamoid.com/resources/guides/flipkart-seller-guide)), so its variant model is \[Unknown\]; Amazon and Walmart are the nearest documented analogues.

## Task 2b: Combos versus plain quantity

Every platform I could read that supports bundles stores the combo as its own sellable, with its own price and identity, and keeps a recipe of components that is multiplied out into component quantities when stock moves; none of them treats a combo as just “quantity × component” on the shelf. I found no public description of how Swiggy, Blinkit or Instamart do this in their dark stores \[Unknown\]; the closest documented analogues are Odoo, Medusa and Shopify.

### Separate SKU or recipe?

Both. The combo is a record with its own price, and the recipe is a list of component-and-quantity rows attached to it.

| Platform | Combo identity and price | Recipe | Where it is expanded | Tag and source |
| --- | --- | --- | --- | --- |
| Shopify fixed bundle | modeled as a product; the parent variant's price determines the price | `bundleComponents` (product, quantity, option selections) and, on the variant, `productVariantComponents`; at most 30 components and 3 options, no nested bundles | the component variants' inventory determines the bundle's inventory; on an order the component lines reference a `LineItemGroup` that carries the bundle's quantity, title and variant | \[Confirmed\] [fixed bundle guide](https://shopify.dev/docs/apps/build/product-merchandising/bundles/add-product-fixed-bundle), [LineItemGroup](https://shopify.dev/docs/api/admin-graphql/2024-10/objects/LineItemGroup), [ProductVariant](https://shopify.dev/docs/api/admin-graphql/latest/objects/ProductVariant) |
| Odoo kit | the kit product | a `mrp.bom` of type phantom (shown as “Kit”) with component lines | when the stock move is confirmed: the kit's own move is dropped and one move per component is created, each labelled with the kit's name for the picker | \[Confirmed\] [mrp\_bom.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/mrp/models/mrp_bom.py#L29), [action\_explode](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/mrp/models/stock_move.py#L378) |
| Odoo combo | a product of type combo (`combo_ids`) | `product.combo` (name, items, `base_price`) and `product.combo.item` (product, `lst_price`, `extra_price`) | at the sale line: the combo line and its item lines are linked through `linked_line_id` and `combo_item_id` | \[Confirmed\] [product\_combo.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_combo.py), [product\_combo\_item.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_combo_item.py) |
| Magento bundle | a product with `type_id` bundle | options (`required`, position, type), each with selections (product, default flag, price type and value, quantity, can-change-quantity) | order rows carry a `parent_item_id` column | \[Confirmed\] for the schema ([options](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Bundle/etc/db_schema.xml#L10), [selections](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Bundle/etc/db_schema.xml#L54)); \[Inferred\] that child order rows use `parent_item_id` |
| Medusa inventory kit | the variant | `required_quantity` (decimal, default 1) on the link between a variant and each inventory item | on purchase, the linked inventory items' quantities are updated | \[Confirmed\] [inventory kit docs](https://docs.medusajs.com/resources/commerce-modules/inventory/inventory-kit), [link definition](https://github.com/medusajs/medusa/blob/f7add009dd2f142d98090a5194750e71b2147ddf/packages/modules/link-modules/src/definitions/product-variant-inventory-item.ts#L11) |
| Medusa bundle recipe | a product for the bundle, whose variant has `manage_inventory` off | custom `Bundle` and `BundleItem` (quantity) models linked to products | at add-to-cart: one line item per component, each carrying the bundle id in metadata | \[Confirmed\] [bundled products recipe](https://docs.medusajs.com/resources/recipes/bundled-products/examples/standard) |
| Amazon virtual bundle | 2 to 5 different complementary products sold together as a set, at a total no higher than the items bought separately; the sets are not packaged as one unit | not described | Amazon packs and ships the sets; the announcement says bundles use existing FBA inventory; how counts are tracked against the individual products is not stated | \[Confirmed\] [selling blog](https://sell.amazon.com/blog/product-bundling), [announcement](https://sellercentral.amazon.com/seller-forums/discussions/t/aeff34dfb538fbb69e2f2af22af23fec); \[Unknown\] for the counting |
| Amazon physical bundle | the seller packages the items as one unit and creates a new listing with its own ASIN | none, because it is a new physical product | stock is counted as that new product | \[Confirmed\] [selling blog](https://sell.amazon.com/blog/product-bundling) for the listing and ASIN; \[Inferred\] for the stock |

Saleor and Spree have no bundle concept in the code I searched: no match for “bundle” or “subscription” in Saleor's product, order and warehouse models, and no bundle-, kit- or assembly-named files in Spree's core app, lib and db folders \[Confirmed for the folders searched\].

### From combo quantity to loose packets

Ordering two of a combo multiplies the recipe. Odoo computes a factor (the ordered quantity converted to the BoM's unit, divided by the BoM's quantity), scales each component line by it, and labels the component moves with the kit's name \[Confirmed: [stock\_move.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/mrp/models/stock_move.py#L378)\]. Medusa's inventory kit does the same job with `required_quantity`; its docs use a bicycle that consumes two wheel items per bicycle sold \[Confirmed: docs\]. Medusa's bundle recipe sets each component line's quantity to the bundle item's quantity times the bundle quantity \[Confirmed: docs\].

```sql
sellable         (sku PK, name, price, is_bundle)
bundle_component (bundle_sku FK, component_sku FK, qty, PRIMARY KEY (bundle_sku, component_sku))   -- the recipe
order_line       (order_id, sku, qty, unit_price)                                                    -- one line for the combo
pick_task        (order_id, component_sku, qty, source_bundle_sku)                                   -- derived, one row per component
```

```text
Combo CHIPS-X3 = 3 x CHIPS-52G, priced Rs 55 (three singles would be Rs 60)
Customer orders CHIPS-X3 with quantity 2
Bill:  one line, 2 x Rs 55 = Rs 110
Pick:  CHIPS-52G, qty = 2 x 3 = 6, source = CHIPS-X3
Stock: CHIPS-52G on hand minus 6; CHIPS-X3 availability = floor(CHIPS-52G available / 3)
```

This is an illustration built from the Odoo and Medusa mechanics above, not any company's implementation.

### Why not offer only quantity?

1. **The price is not quantity × the single price.** A Shopify bundle takes its price from the parent variant; Magento selections carry a price type and value; Odoo combos carry a `base_price` and a per-item `extra_price` \[Confirmed: docs and fields\]. The combo needs somewhere to hold its own price rule \[Inferred\].
2. **The combo is something customers find and buy.** Shopify models a fixed bundle as a product \[Confirmed\], which gives it its own page, image and options \[Inferred\].
3. **Discounts and promotions.** Amazon's selling guide says a bundle typically costs less than buying the items individually, that bundles encourage customers to buy more in one transaction, and that sellers can run bundle promotions; for virtual bundles the total must be equal to or less than the items' combined individual prices \[Confirmed: [selling blog](https://sell.amazon.com/blog/product-bundling)\]. Amazon's [announcement](https://sellercentral.amazon.com/seller-forums/discussions/t/aeff34dfb538fbb69e2f2af22af23fec) likewise describes complementary items bought together at a discount \[Confirmed\].
4. **Moving slow stock.** The same guide calls bundling an effective way to move slow-selling items \[Confirmed\].
5. **Choices inside a combo.** Magento options have a required flag and selections a can-change-quantity flag; Odoo combos list the allowed items with an extra price each \[Confirmed: fields\]. A plain quantity cannot express “pick 3 of these 5” \[Inferred\].
6. **Stock stays at component level.** Shopify derives bundle inventory from the components, Amazon's virtual bundles reuse existing FBA inventory, and Medusa's kit links the variant to the component inventory items \[Confirmed\]. That is what lets a warehouse pick loose packets while the customer buys one combo \[Inferred\].
7. **Known limits.** Medusa's docs say the inventory-kit approach does not support custom bundle pricing or separate fulfillment of a bundle's items \[Confirmed: docs\]; Shopify caps a bundle at 30 components and has no nested bundles \[Confirmed: docs\].

For identical items there is a quantity-only alternative, a pack unit. Odoo's product template carries a list of packagings (`uom_ids`) \[Confirmed: [product\_template.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_template.py)\], so “pack of 3” can be a unit of the same item. It avoids a recipe but cannot mix different items \[Inferred\].

## Task 2c: Units of measure

Platforms store one base unit per item and every other unit as a factor to it; what varies is how much of that is a real table (Odoo has one, Magento and Medusa keep only a flag or free text) and how they handle items whose billed quantity is only known at pick time (Instacart documents this most clearly). I found no public description of how Swiggy, Blinkit or Instamart store units \[Unknown\].

### Base-unit conversion

| Platform | How units are stored | Conversion | Tag and source |
| --- | --- | --- | --- |
| Odoo 19 | a `uom.uom` table: `name`, `relative_uom_id` (the reference unit), `relative_factor` (labelled “Contains”) and a computed, stored `factor`; default units include a dozen and a pack of 6; a product has a default stock unit `uom_id` and extra units in `uom_ids`; sale and invoice lines carry their own `product_uom_id` | `factor = relative_factor × relative_uom_id.factor`; one global rounding precision (“Product Unit”) shared by all units | \[Confirmed\] [uom\_uom.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/uom/models/uom_uom.py#L34), [product\_template.py](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/product/models/product_template.py) |
| Medusa v2 | `InventoryItem.unit_of_measure`, a nullable free-text column; the variant-to-inventory-item link has a decimal `required_quantity` | none in the inventory models I read | \[Confirmed\] [inventory-item.ts](https://github.com/medusajs/medusa/blob/f7add009dd2f142d98090a5194750e71b2147ddf/packages/modules/inventory/src/models/inventory-item.ts#L13); the missing conversion is \[Inferred\] from what I read |
| Magento 2 | no unit table in the eight module schemas I checked; stock quantity can be decimal (`is_qty_decimal`) with `qty_increments` and `enable_qty_increments`; the order item repeats `is_qty_decimal` | none | \[Confirmed\] [stock item](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/CatalogInventory/etc/db_schema.xml#L23), [order item](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Sales/etc/db_schema.xml#L465) |
| Shopify | a variant carries a `unitPriceMeasurement` | five fields: `measuredType`, `quantityUnit`, `quantityValue`, `referenceUnit`, `referenceValue` | \[Confirmed\] [ProductVariant](https://shopify.dev/docs/api/admin-graphql/latest/objects/ProductVariant), [UnitPriceMeasurement](https://shopify.dev/docs/api/admin-graphql/latest/objects/UnitPriceMeasurement) |
| Instacart | each product has a `cost_unit` of each, lb or kg | weighted items add a catalog `par_weight` (see below) | \[Confirmed\] [Instacart docs](https://docs.instacart.com/connect/fulfillment_guide/concepts/product_quantity) |

### Price per unit

Shopify's pair of measurements is the cleanest public model. One half says what is in the pack (`quantityValue` and `quantityUnit`), the other says what to show the price per (`referenceValue` and `referenceUnit`) \[Confirmed: field names\]. The displayed price is then price × referenceValue ÷ quantityValue after converting both to one unit, so Rs 100 for 500 g shown per 100 g is Rs 20 \[Inferred: the formula and the example are mine\].

GS1's data-sync network has a matching attribute, `netContent`, defined as “the amount of the trade item contained by a package, usually as claimed on the label” and written as a value with a unit, for example 750 MLT or 20 ea. Its rules: use the label claim for fixed-content items, the average quantity for variable-quantity items, and the total for multi-packs \[Confirmed: [GS1 Germany attribute page](https://gdsn.gs1-germany.de/BMSattribute?id=3733)\].

### Weighted items

Instacart documents three modes, which is the clearest public treatment of “sold by kg, charged by actual weight”:

| Mode | `cost_unit` | Quantity field | Docs wording |
| --- | --- | --- | --- |
| Per each | `each` | `count` | “Items of a fixed size and price per unit.” |
| By weight | `lb` or `kg` | `weight` | “Loose items purchased and priced by weight.” |
| Variable weight | `lb` or `kg`, plus `par_weight` in the catalog | `count` or `weight` | “Items purchased as a unit, but priced by weight.” If `count` is missing, the system calculates it from `par_weight`. |

&#91;Confirmed: [Instacart product quantity docs](https://docs.instacart.com/connect/fulfillment_guide/concepts/product_quantity)\]. GS1 calls the general case a variable measure trade item: one that is “priced, ordered or invoiced in quantities which can vary”, such as fruit and vegetables, meat, cheese or fabric; for retail items either the price or the quantity must be encoded in the barcode \[Confirmed: [GS1 support article](https://support.gs1.org/support/solutions/articles/43000734396-what-is-a-variable-measure-trade-item-)\].

Ordered, fulfilled and charged quantities are kept apart. An Odoo sale line has `product_uom_qty`, `qty_delivered`, `qty_invoiced` and `qty_to_invoice`; a Magento order item has `qty_ordered`, `qty_shipped`, `qty_invoiced`, `qty_refunded` and `qty_canceled` \[Confirmed: columns\]. Separate counters are what let a line be ordered at 1 kg and invoiced at 1.07 kg \[Inferred: none of the docs I read walks through a weighed example\].

```text
Order:    1 kg tomatoes at Rs 40 per kg  ->  qty_ordered 1.000 kg, unit_price Rs 40.00 per kg
Picking:  the picker weighs 1.07 kg     ->  qty_fulfilled 1.070 kg
Invoice:  charge the actual weight       ->  qty_charged 1.070 kg x Rs 40.00 = Rs 42.80
```

That is an illustration, not any company's flow. How Blinkit or Instamart price loose produce is \[Unknown\].

### Standards

UN/CEFACT Recommendation 20 provides three-character alphabetic and alphanumeric codes for units of measurement, meant for systems that exchange trade information \[Confirmed: [UNECE](https://unece.org/trade/documents/2021/06/uncefact-rec20-0)\]; for example C62 is “one”, a synonym for unit \[Confirmed: [UNECE vocabulary](https://service.unece.org/trade/uncefact/vocabulary/rec20/)\]. GS1 supplies `netContent` and the variable-measure rules above.

Odoo is the one open-source core where I found UN/CEFACT codes inside the commerce code itself. Its accounting module keeps a dictionary from unit-of-measure records to Recommendation 20 codes (`unit` → `C62`, `dozen` → `DZN`, `kg` → `KGM`, `g` → `GRM`, `litre` → `LTR` … 28 units in all). A unit with no entry falls back to `C62` (“one”), and a reverse lookup, `_get_uom_from_unece_code`, falls back to the plain `unit` record. The values Odoo prepares for an invoice line include `unece_uom_code`, taken from the product's own unit. \[Confirmed\] [uom\_uom.py#L6](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/account/models/uom_uom.py#L6), [account\_move\_line.py#L3550](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/account/models/account_move_line.py#L3550). I searched only Odoo's code for UN/CEFACT codes, so whether Saleor, Medusa, Magento or Spree map their units to them is \[Unknown\].

### A simplified schema

```sql
uom        (uom_id PK, code, name, base_uom_id FK, factor_to_base NUMERIC)               -- g -> kg 0.001, dozen -> piece 12
item       (item_id PK, name, base_uom_id FK, net_content_value, net_content_uom_id FK)   -- a 500 g pack
order_line (line_id PK, item_id FK, uom_id FK, qty_ordered, qty_fulfilled, qty_charged, unit_price)
```

Simplified from Odoo's unit table and the ordered, delivered and invoiced counters on its sale lines.

## Task 3: Billing across physical items and subscriptions

In every system I could read, the invoice or order line is its own row that stores quantity, unit price and tax as they were at the time of sale, and the type-specific pricing rule (quantity × rate for goods, plan × duration for subscriptions) runs once to produce those numbers instead of being recomputed from the live catalog later. \[Confirmed\] for Saleor, Medusa, Magento, Odoo, Spree, Shopify and Stripe; \[Unknown\] for Amazon, whose billing internals are not public.

### 3.1 How lines point at different kinds of items

There are two shapes. Most commerce platforms use one line table with a nullable pointer to the catalog plus a set of copied columns. Stripe splits billing out: an invoice line points at whatever generated it, either a one-off invoice item or a subscription item.

| Platform | Pointer back to the catalog | What the line stores itself | Tag and source |
| --- | --- | --- | --- |
| Saleor | `variant` foreign key, nullable, `on_delete=SET_NULL` | product and variant names, SKU, product-type id, `is_shipping_required`, `is_gift_card`, currency, unit and total price (net and gross), undiscounted prices | \[Confirmed\] [OrderLine](https://github.com/saleor/saleor/blob/5acd08ffe7389d7323344dd3e1398d8a46367919/saleor/order/models.py#L567) |
| Medusa v2 | `variant_id` and `product_id` as nullable text ids | titles, `variant_sku`, `variant_barcode`, option values (json), `requires_shipping`, `is_giftcard`, `unit_price`; tax lines and adjustments are separate rows | \[Confirmed\] [line-item.ts](https://github.com/medusajs/medusa/blob/f7add009dd2f142d98090a5194750e71b2147ddf/packages/modules/order/src/models/line-item.ts#L11) |
| Magento 2 | `product_id`, `product_type`, `product_options`; invoice items point at `order_item_id` | `sku`, `name`, `description`, `price`, `original_price`, tax and discount columns, `row_total`, `is_virtual`, `is_qty_decimal` | \[Confirmed\] [sales\_order\_item](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Sales/etc/db_schema.xml#L465), [sales\_invoice\_item](https://github.com/magento/magento2/blob/16f4fa211bfdecda84c3e7d475f1514780b170c6/app/code/Magento/Sales/etc/db_schema.xml#L1156) |
| Odoo 19 | `product_id` on both the sale line and the invoice line; the invoice line links back through `sale_line_ids` | `name`, `quantity`, `product_uom_id`, `price_unit`, `discount`, `tax_ids`, stored `price_subtotal` and `price_total` | \[Confirmed\] [account.move.line](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/account/models/account_move_line.py), [\_prepare\_invoice\_line](https://github.com/odoo/odoo/blob/807b2abc274e7497ce93747ba2d3759f32e88250/addons/sale/models/sale_order_line.py#L1529) |
| Spree | `belongs_to :variant`, loaded `with_deleted` | price and tax category are copied when the line is created; name and SKU are read through the variant | \[Confirmed\] for the code ([line\_item.rb](https://github.com/spree/spree/blob/a0e5b66419569d1e91d3b1eaf180444b73c65cd1/spree/core/app/models/spree/line_item.rb#L18)); \[Inferred\] that nothing else is copied, because I did not read the database schema |
| Shopify | `variant` and `product` are nullable | `title` as of order creation, `variantTitle`, `originalUnitPriceSet`, `taxLines` | \[Confirmed\] [LineItem](https://shopify.dev/docs/api/admin-graphql/latest/objects/LineItem) |
| Stripe | `parent.type` is `invoice_item_details` or `subscription_item_details` | `amount`, `quantity`, `description`, `period.start` and `period.end`, `pricing`, `taxes`, `discount_amounts` | \[Confirmed\] [Invoice line item](https://docs.stripe.com/api/invoice-line-item/object) |

The pointer is nullable on purpose in at least two of these. Saleor's `SET_NULL` lets a variant be deleted while old order lines keep their copied name and SKU \[Confirmed: code\]; the code does not say why, and I read it as a design choice that lets the catalog change without rewriting history \[Inferred\]. Shopify's docs mark `variant` and `product` as nullable without giving a reason \[Confirmed: docs\]; a [community answer](https://community.shopify.com/t/graphql-admin-api-orders-query-null-lineitem-variant/366644) says the field is null when the product was deleted \[Inferred: secondary source\]. Spree goes the other way: variants are soft-deleted (`acts_as_paranoid`, see [variant.rb](https://github.com/spree/spree/blob/a0e5b66419569d1e91d3b1eaf180444b73c65cd1/spree/core/app/models/spree/variant.rb#L5)) and the line loads them `with_deleted`, so the pointer keeps resolving \[Confirmed: code; the intent is my reading\].

Odoo also shows how a bundle looks on an invoice: when the sale line is a combo product, its invoice line becomes a section header titled with the product name and quantity (`<name> x <qty>`) \[Confirmed: `_prepare_invoice_line`\]. The chosen items then follow as ordinary priced lines \[Inferred: only the combo product takes the section branch in that method\].

### 3.2 How each type's amount is computed and reconciled

The line does not need to know the pricing rule, only its result. What differs by type is the rule that runs while the line is being created:

| Item type | Rule that produces the line amount | Where the inputs live | Tag and source |
| --- | --- | --- | --- |
| Physical goods | quantity × unit price, then discounts, then tax | the line keeps quantity, unit price and the resulting totals (Saleor keeps unit and total price in net and gross, plus undiscounted values) | \[Confirmed\] for the columns; \[Inferred\] for the order of operations |
| Subscription | one price per plan and billing interval; amount = unit amount × quantity for that period; the line carries the period start and end | Stripe Price has `recurring.interval` and `interval_count`; `interval=month` with `interval_count=3` bills every 3 months | \[Confirmed\] [Prices](https://docs.stripe.com/api/prices/object), [Invoice line item](https://docs.stripe.com/api/invoice-line-item/object) |
| Tiered or metered | quantity runs through tiers (`graduated` or `volume`) or comes from reported usage | Stripe `billing_scheme`, `tiers_mode`, `usage_type` | \[Confirmed\] [Prices](https://docs.stripe.com/api/prices/object) |

Your Prime example maps onto that model as two plans × four intervals, eight price rows, not one monthly price multiplied by the number of months \[Inferred: my mapping onto Stripe's documented fields; the docs do not use this example\]. The worked example below stores one price per plan and duration for the same reason: longer terms are usually discounted.

A subscription does not have to share the product line table. The three systems below document three different ways of attaching one to an order.

| System | What the subscription is | How it reaches the order or invoice | Tag and source |
| --- | --- | --- | --- |
| Stripe Billing | A `Subscription` over recurring `Price`s (`recurring.interval`, `recurring.interval_count`). The same `Price` object also prices one-time goods: its `type` is `one_time` or `recurring` | An invoice line's `parent` (nullable) names “the parent that generated this line item”, and its `period` is “the subscription period” for subscription lines | \[Confirmed\] [Price](https://docs.stripe.com/api/prices/object), [Invoice line item](https://docs.stripe.com/api/invoice-line-item/object) |
| Shopify | A `SellingPlan` (“How a product can be sold and purchased through recurring billing or deferred purchase options.”) whose `billingPolicy` can be recurring, with `interval` (day, week, month or year), `intervalCount`, `minCycles` and `maxCycles` | The order line item has a nullable `sellingPlan` field (“The selling plan details associated with the line item.”), so a subscription purchase is an ordinary order line plus those plan details | \[Confirmed\] [SellingPlan](https://shopify.dev/docs/api/admin-graphql/latest/objects/SellingPlan), [SellingPlanRecurringBillingPolicy](https://shopify.dev/docs/api/admin-graphql/latest/objects/SellingPlanRecurringBillingPolicy), [LineItem](https://shopify.dev/docs/api/admin-graphql/latest/objects/LineItem) |
| Medusa (a documented recipe, not the core) | A custom `Subscription` data model with an `interval` (monthly or yearly) and a `period`: “For example, if period is 3 and interval is monthly, then a new order is created every three months.” | A scheduled job hands each due subscription to a workflow that builds the order from “the cart originally used to make the subscription purchase” and captures its payment; the model is linked to the cart, the customer and the order | \[Confirmed\] [Subscriptions recipe](https://docs.medusajs.com/resources/recipes/subscriptions/examples/standard) |

&#91;Inferred\] In Shopify and Medusa the subscription is a layer around orders: the Shopify line item only holds the plan's details, and the Medusa workflow creates a new order each cycle. In Stripe it is a billing object whose output is invoice lines. In all three the paid line still stores its own amount (Stripe's invoice line has `amount`, `period` and `pricing`), so the worked example below also copies plan, duration, period and price onto its line.

Mixed baskets work by accepting both kinds of price in one request. Stripe Checkout in subscription mode takes recurring and one-time prices together; the one-time items “will be on the initial invoice only”, with limits of 20 recurring and 20 one-time lines \[Confirmed: [Checkout Sessions](https://docs.stripe.com/api/checkout/sessions/create)\]. I found no public description of how Amazon combines a Prime plan and a physical item in one charge \[Unknown\].

Reconciliation is plain arithmetic over lines: header net is the sum of line net, header tax is the sum of line tax, and gross is net plus tax. The example below derives the header from the lines and asserts it. Real systems add rounding rules (round each line, or round the invoice) that differ by jurisdiction, and I did not research them \[Unknown\].

None of the open-source cores I read model subscriptions. Searching Saleor's product, order and warehouse models for “subscription” and “bundle” found nothing, Spree's core app, lib and db folders have no files named for subscriptions or bundles, and the Odoo 19 community checkout has no subscription addon \[Confirmed for the folders searched\]. That fits subscriptions living in a separate billing system, as in Stripe \[Inferred\].

### 3.3 Why the invoice copies data instead of reading the live catalog

1. **Tax law lists what an invoice must show.** India's [CGST Rule 46](https://taxinformation.cbic.gov.in/content/html/tax_repository/gst/rules/cgst_rules/active/chapter6/rule46_v1.00.html) requires, among other particulars, the description of goods or services (clause h), quantity and unit (i), total value (j), taxable value after discount (k), tax rate (l) and HSN code (g) \[Confirmed\]. The rule says what must appear, not how to store it; treating that as a reason to copy the values onto the invoice is my reading \[Inferred\].
2. **Stripe locks an invoice once it is finalized.** Invoices are editable only as drafts; after finalization “you can’t change most of its details, including fields related to the amount or customer”, to satisfy the tax-compliance requirement that finalized invoices be kept as they were finalized, and amount changes go through credit notes \[Confirmed: [Stripe docs](https://docs.stripe.com/invoicing/integration/workflow-transitions)\].
3. **Catalog rows get renamed, re-priced and deleted.** The nullable pointers and soft deletes in 3.1 are how three platforms keep history intact while the catalog changes \[Confirmed: code; the motive is Inferred\].
4. **Re-pricing from the live catalog gives the wrong total.** The example below shows it with numbers.

### 3.4 Worked example: 3 packets of chips plus a 3-month video plan

This is an illustration I built and ran in SQLite on 22 Oct 2026, not a description of any company's system. Money is stored in paise and tax in basis points; the prices, plans and tax rates are invented.

```sql
CREATE TABLE product    (product_id INTEGER PRIMARY KEY, sku TEXT UNIQUE, name TEXT, unit_price INTEGER, tax_bp INTEGER);
CREATE TABLE plan       (plan_id INTEGER PRIMARY KEY, code TEXT UNIQUE, name TEXT, tax_bp INTEGER);
CREATE TABLE plan_price (plan_id INTEGER REFERENCES plan, duration_months INTEGER, price INTEGER,
                         PRIMARY KEY (plan_id, duration_months));
CREATE TABLE invoice    (invoice_id INTEGER PRIMARY KEY, issued_on TEXT, net INTEGER, tax INTEGER, gross INTEGER);
CREATE TABLE invoice_line (
  line_id INTEGER PRIMARY KEY, invoice_id INTEGER REFERENCES invoice,
  item_type TEXT CHECK (item_type IN ('PRODUCT','SUBSCRIPTION')),
  item_id INTEGER,                          -- polymorphic pointer, deliberately no foreign key
  description TEXT, quantity INTEGER, unit_price INTEGER,
  period_months INTEGER, period_start TEXT, period_end TEXT,   -- NULL for goods
  tax_bp INTEGER, net INTEGER, tax INTEGER, gross INTEGER);
```

The two pricing rules run when the cart is turned into lines:

```text
goods:         net = quantity * product.unit_price
subscription:  net = quantity * plan_price(plan_id, months).price     -- looked up by plan and duration
both:          tax = (net * tax_bp + 5000) // 10000                   -- integer paise, round half up
               gross = net + tax
subscription:  period_start = today, period_end = today + months
```

Prices: chips Rs 20.00 at 12% tax. StreamPlus Video-only is Rs 149, 399, 749, 1399 for 1, 3, 6, 12 months; Full is Rs 299, 799, 1499, 2799; both at 18%. The cart is 3 chips and 1 Video-only for 3 months, issued 2026-10-21.

| Line | Qty | Rate | Net | Tax | Gross |
| --- | --- | --- | --- | --- | --- |
| Chips 52 g, Classic Salted | 3 | Rs 20.00 | Rs 60.00 | Rs 7.20 (12%) | Rs 67.20 |
| StreamPlus Video-only, 3 months (2026-10-21 to 2027-01-21) | 1 | Rs 399.00 | Rs 399.00 | Rs 71.82 (18%) | Rs 470.82 |
| **Invoice total** |  |  | **Rs 459.00** | **Rs 79.02** | **Rs 538.02** |

The assertion gross = net + tax = sum of line gross passes. Then the catalog changes: chips go from Rs 20 to Rs 22 and the 3-month Video-only price from Rs 399 to Rs 449. The stored invoice still reads Rs 538.02. Re-pricing the same lines from the live catalog would give a net of Rs 515.00 instead of the Rs 459.00 actually charged, which is exactly the error the copied columns prevent.

### 3.5 What I couldn't establish for Task 3

- How Amazon bills a subscription next to a physical item, and what its invoice lines look like internally. \[Unknown\]
- How Swiggy, Zomato or Blinkit bill mixed carts. \[Unknown\]
- Tax rounding rules per jurisdiction (line-level or invoice-level). \[Unknown\]

## What I couldn't find

Most gaps are company internals: Swiggy, Zomato, Blinkit, Amazon and Flipkart publish little about their schemas. Where that happened I used the closest documented analogue and said so in the section.

| Gap | What I found instead | Closest analogue used |
| --- | --- | --- |
| Whether Swiggy's Food, Instamart, Bolt and Toing share databases, and how the app presents them as tabs | statements that Bolt is a Food tile, Instamart has its own app, and Toing reuses existing tech and infrastructure | Grab (shared order platform), Uber (edge gateway, shared fulfillment), DoorDash |
| How Bolt listings relate to the Food catalog | press releases describing a curated menu from restaurants already on Swiggy | a filtered view of Food (my inference, Task 1a) |
| Toing's shared login, accounts and database | letter statements and user metrics | none |
| Zomato and Blinkit backends and catalogs | Eternal's letter says the going-out offering moves to a separate District app; nothing on backends | Grab, DoorDash |
| Where Bolt and Instamart sit in the Swiggy app today (Oct 2026) | launch-time placement and the standalone Instamart app | none |
| Instamart's delivery-time promise (the brief says 10 to 15 minutes) | the current App Store listing states no delivery time | none |
| Blinkit's ambulance: operator, payment, cart flow, data model, any official price | Jan 2025 press reports (Business Today, MediaNama, YourStory): bookable in the app, staffed by a paramedic, an assistant and a driver, with an unofficial price starting at Rs 2,000 | Odoo's service type, Uber's layering |
| Amazon's internal catalog storage; how gift cards, Prime plans and digital items are modeled; invoice line internals | the public catalog and product-type APIs and the bundling rules | Magento and Saleor product types; Stripe billing |
| Flipkart's variant model | only third-party integration guides; the Flipkart seller API page I opened covers sign-up and OAuth only | Amazon and Walmart variant groups |
| How dark stores expand combos into picks and deduct stock | nothing public | Odoo kits, Medusa inventory kits, Shopify bundles |
| How quick-commerce platforms price and charge loose items | nothing public | Instacart's weighted-item modes |
| Odoo's subscription implementation | the community code has no subscription addon, and the docs page I opened did not return its body | Stripe, Shopify selling plans, Medusa's subscription recipe |
| Tax rounding rules per jurisdiction | not researched | none |

### How the evidence was gathered, and its limits

Open-source code was read from local clones at the commits linked in each citation. Web pages were read through a fetch tool that returns summaries, so I quote only short phrases and link every claim so you can check it. A few pages were unusable: Swiggy Bytes' serviceability post blocks automated readers (I did not work around that), Amazon Seller Central help pages need a login, the GS1 Navigator page needs JavaScript (I used GS1 Germany's copy of the GDSN attribute instead), and Amazon's Orders API model page would not load.
