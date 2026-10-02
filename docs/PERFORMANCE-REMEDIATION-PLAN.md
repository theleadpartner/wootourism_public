# WooTourism Performance Remediation Plan

**Repository:** `theleadpartner/wootourism_public`  
**Baseline branch:** `main`  
**Baseline HEAD:** `107bd36eccb87de265126661726c8232750bf1da`  
**Date:** 2026-10-02  
**Status:** Documentation only. A matching WooTourism source repository has now been identified in `theleadpartner/wootourism`, but implementation remains blocked in this repository because `wootourism_public` contains no runtime source and the private repository is outside the authorized write scope of this remediation.

## 1. Purpose

Document the safe work required to determine whether WooTourism contributes to the reported All Ways Colombia production symptoms:

- a `wp-content/debug.log` reported at approximately 698 MB;
- a frontend TTFB measurement reported around 2.244 seconds.

The operational measurements are external observations and are not stored in this repository.

## 2. Repository state verified

On the current `main` branch:

- the repository contains only `README.md`;
- `README.md` contains only `# wootourism`;
- no PHP, JavaScript, CSS or build/runtime source exists;
- `specs/README.md` does not exist;
- no architecture SPEC exists;
- no Quality Gates document exists;
- no UI/Design System tree exists;
- no PENDING convention exists.

Therefore this repository cannot currently be used to inspect or modify the production WooTourism implementation.

## 3. Source-of-truth constraint

No WooTourism implementation may be created from assumptions in this repository.

Read-only verification on 2026-10-02 identified `theleadpartner/wootourism` as a private repository containing WooTourism version `1.0.2`, matching the version and diagnostic strings observed in production. Its verified `main` HEAD at the time of review was `45bdae73168c499a7f39dfc8bf7f687d6fb0ba8a`.

The private bootstrap also contains the production-relevant pattern:

- `WOOTOURISM_DEBUG` defaults to `true` when not predefined;
- normal plugin initialization is wrapped in debug logging;
- loaded files and initialized components are logged;
- additional runtime paths use `error_log()` when that debug flag is enabled.

This strongly identifies the private repository as the matching implementation source, but it does **not** prove that production is deployed from that exact commit. Server-side deployment metadata would be needed to establish the precise deployed SHA.

The requested implementation scope authorizes writes only to `theleadpartner/all_ways_colombia` and `theleadpartner/wootourism_public`. Therefore the private source was reviewed read-only and was not modified.

Until repository ownership for the remediation is explicitly decided, `wootourism_public` remains documentation-only.

## 4. Production evidence recorded 2026-10-02

A production audit supplied for All Ways Colombia confirms that the deployed WooTourism runtime is actively producing high-volume diagnostic output. This evidence is operational and does **not** substitute for canonical source code in this repository.

Observed in production:

- WooTourism reports itself as version `1.0.2`.
- The WordPress `debug.log` was approximately 699 MB with more than 14 million lines.
- More than 6.8 million historical log matches contained `WooTourism`.
- Recent normal requests emitted repeated initialization messages such as plugin initialization, loaded-file tracing, frontend component initialization, checkout handler initialization and AJAX handler initialization.
- Repeated order-status array dumps and per-product "forcing product as purchasable" messages were among the highest-frequency recent patterns.
- `WP_DEBUG` and `WP_DEBUG_LOG` were enabled in the serving WordPress configuration at the time of the audit.

The production log also reveals filenames/classes such as `class-wootourism-pricing.php`, `class-wootourism-cart.php`, `class-wootourism-ajax-handler.php`, `class-wootourism-order-status.php` and several frontend/dashboard classes. These names are diagnostic evidence only; their implementations are not present here and must not be reconstructed from log strings.

### Required remediation once canonical source is available

The first WooTourism implementation PR should, before broader optimization:

1. remove or production-gate normal-success-path `error_log()` tracing and array dumps;
2. preserve genuine warnings/errors needed for operational diagnosis;
3. verify whether a plugin-level debug constant exists and ensure production defaults do not force verbose logging;
4. preserve booking, cart, checkout, order-status, AJAX and pricing contracts;
5. measure log growth and TTFB again after deployment before changing pricing or availability architecture.

## 5. Known integration contract from All Ways Colombia

The companion repository currently reads WooTourism metadata through `wc-dynamic-search-menu/includes/wootourism-price-helper.php`.

The observed keys include:

- `_wootourism_booking_type`
- `_wootourism_accommodation_price_per_night`
- `_wootourism_restaurant_price_per_seat`
- `_wootourism_tour_price_per_spot`
- `_wootourism_special_price_per_spot`

The helper currently recognizes booking types:

- `accommodation`
- `restaurant`
- `tour`
- `special`

These values are integration assumptions from the companion repository, not proof of WooTourism's canonical implementation. They must be verified against the real WooTourism source before changing either side.

## 6. Mandatory consolidation strategy once source code exists

### REUTILIZAR

Use the actual WooTourism price/storage owner and existing hooks as the source of truth.

### REFACTORIZAR / EXTENDER

Optimize the canonical price access/invalidation path in place if profiling proves WooTourism work is expensive.

### SUSTITUIR

Only replace an internal implementation when all current consumers can remain on the same public contract and the superseded path can be removed safely.

### CREAR

Do not create parallel pricing metadata, duplicate price providers, shadow caches, new endpoints, cron, workers or compatibility plugins solely to work around the missing source.

## 7. Audit plan after the canonical source is available

### Phase WT-1 — map ownership

Identify and document:

- plugin bootstrap
- product type/booking type owner
- price owner for each booking type
- save/update hooks
- product-meta write paths
- AJAX endpoints
- REST endpoints
- cron/background jobs
- external providers
- frontend asset loading
- product-page/catalog consumers
- compatibility/legacy code

### Phase WT-2 — verify the price contract

Confirm whether the metadata read by All Ways Colombia is:

- canonical persistent product data;
- derived compatibility data;
- legacy data;
- or mirrored data.

No optimization may change price semantics until this is known.

### Phase WT-3 — profile expensive paths

Specifically inspect:

- repeated `WC_Product` construction;
- N+1 post-meta access;
- unbounded product queries;
- callbacks on `init`, `wp_loaded`, `template_redirect`, `wp_enqueue_scripts` or product filters;
- synchronous external calls;
- expensive availability/calendar computation;
- logging on normal requests;
- cache invalidation storms.

### Phase WT-4 — coordinate with All Ways Colombia

The companion remediation must preserve WooTourism's canonical price contract.

If a derived range cache is introduced in `all_ways_colombia`, WooTourism must expose or already have a reliable invalidation signal for all relevant price mutations. If it does not, that decision requires explicit approval before a persistent cache is merged.

## 8. Verified private-source audit for the first implementation PR

Read-only inspection of `theleadpartner/wootourism@45bdae73168c499a7f39dfc8bf7f687d6fb0ba8a` identified the following high-priority logging owners:

| Owner | Observed `error_log()` calls | Notes |
| --- | ---: | --- |
| `wootourism.php` | multiple | Bootstrap defaults `WOOTOURISM_DEBUG` to `true` and logs normal initialization/load/component success paths. |
| `includes/class-wootourism-cart.php` | 45 | Mostly debug-guarded, but the default-true flag makes cart/checkout/product hooks noisy in production; includes `print_r()` dumps. |
| `includes/class-wootourism-ajax-handler.php` | 51 | Highest-risk logging owner: many request-path logs are not guarded by `WOOTOURISM_DEBUG`, including `$_POST`, request headers, nonce flow, booking payloads, pricing values and stack traces. |
| `includes/class-wootourism-checkout-fields.php` | 8 | Debug-guarded lifecycle and save tracing. |
| `includes/class-wootourism-order-status.php` | 2 | Debug-guarded initialization and order-status array dump. |

Other reviewed owners:

- `includes/class-wootourism.php`: canonical product/booking hooks and price filters; no `error_log()` found in the reviewed file.
- `includes/class-wootourism-pricing.php`: pricing owner; no `error_log()` found in the reviewed file.
- `includes/class-wootourism-booking-types.php`: no `error_log()` found in the reviewed file.
- `includes/class-wootourism-availability.php`: owns `wp_ajax_wootourism_check_availability` / nopriv equivalent; no `error_log()` found in the reviewed file.

### Canonical contracts to preserve during remediation

The first implementation PR in the canonical WooTourism owner must preserve at minimum:

- existing WooCommerce booking/product price hooks;
- cart-item booking data and calculated-price behavior;
- `woocommerce_before_calculate_totals` pricing path;
- order-line booking persistence;
- booking database writes and status updates;
- payment-gateway filtering;
- order-status semantics;
- `wp_ajax_wootourism_calculate_price` and guest equivalent;
- `wp_ajax_wootourism_check_availability` and guest equivalent;
- checkout identification fields;
- existing custom-table schemas and option/version ownership.

### Authorized remediation design once WT-PENDING-001 is resolved

Use the existing owners only:

1. change the bootstrap default so production does not force verbose debug logging;
2. remove normal-success-path tracing from bootstrap, cart, checkout fields and order-status paths;
3. remove request/payload/header/response dumps from the AJAX pricing handler;
4. retain bounded logging only for genuine exceptional failures where operationally useful;
5. do not change AJAX actions, nonces, response shapes, booking/pricing semantics, checkout data or database ownership;
6. bump WooTourism plugin version once so deployment can be verified unambiguously;
7. validate that the diff contains no pricing, booking, checkout or persistence behavior change;
8. after deployment, rotate/truncate the historical log and remeasure log growth and TTFB before considering wider optimization.

No new logger, endpoint, cache, cron, worker, table or compatibility layer is needed for this remediation.

## 9. Runtime changes authorized by this documentation PR

None.

This PR must not add:

- PHP runtime
- WordPress hooks
- post meta
- options
- transients
- tables
- REST/AJAX routes
- cron
- queues/workers
- frontend JS/CSS
- production code copied from another repository

## 10. Validation gates for a future implementation PR

Once the actual source is present, any remediation PR must confirm:

- source-of-truth owner for price and booking type;
- no change to stored price semantics without explicit approval;
- no duplicate price owner;
- no new N+1 query path;
- no normal-request debug logging;
- no unbounded query introduced;
- no parallel cache without explicit invalidation;
- current product booking flows remain functional;
- complete branch diff contains only authorized remediation changes.

## 11. PENDING and blockers

### BLOCKER-WT-001 — runtime source absent from the authorized public repository

`wootourism_public` still contains no PHP/JS/CSS runtime. The matching private implementation has been identified, but copying private source into a public repository would be a separate publication/governance decision and is not authorized by this remediation.

### WT-PENDING-001 — choose the canonical write target for WooTourism remediation

**Decision required before code changes:**

- **Option A — recommended:** keep `theleadpartner/wootourism` as the canonical runtime owner and explicitly authorize remediation there. This avoids publishing private source and preserves the existing owner/source-of-truth relationship.
- **Option B:** intentionally synchronize approved WooTourism source into `wootourism_public` and make that repository the canonical owner. This requires an explicit decision about public exposure, synchronization ownership and retirement of the previous private source to avoid dual truth.

No code dependent on this decision is implemented in either repository.

### Other PENDING

None.

If a later audit discovers a decision involving a new persistent cache, new endpoint, new background job or changed price semantics, that specific decision must be documented and approved before implementation.

## 12. Completion criteria

WooTourism's part of this remediation can only be considered complete when:

1. `WT-PENDING-001` is resolved and a single canonical write target is authorized;
2. the exact deployed source/version is verified where operationally possible;
3. owner/source-of-truth and price contracts are verified;
4. high-volume normal-request logging is removed or production-gated in the canonical owner;
5. profiling confirms or excludes WooTourism as a material contributor to the reported TTFB;
6. any necessary code changes are implemented in the canonical owner rather than duplicated in this placeholder repository;
7. the integration with `theleadpartner/all_ways_colombia` is revalidated.
