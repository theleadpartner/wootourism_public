# WooTourism Performance Remediation Plan

**Repository:** `theleadpartner/wootourism_public`  
**Baseline branch:** `main`  
**Baseline HEAD:** `107bd36eccb87de265126661726c8232750bf1da`  
**Date:** 2026-10-02  
**Status:** Documentation only. Implementation remains blocked because the repository does not currently contain WooTourism source code. Production evidence gathered on 2026-10-02 increases the priority of resolving this blocker.

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

Before any code remediation:

1. identify the repository or branch that contains the exact WooTourism code deployed on All Ways Colombia;
2. confirm its current production version/commit where possible;
3. decide whether that canonical source should be synchronized into this repository or whether remediation belongs in a different repository;
4. compare the deployed/canonical implementation with the integration assumptions currently present in `theleadpartner/all_ways_colombia`.

Until that source-of-truth question is resolved, this repository remains documentation-only for this issue.

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

## 8. Runtime changes authorized by this documentation PR

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

## 9. Validation gates for a future implementation PR

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

## 10. PENDING and blockers

### BLOCKER-WT-001 — canonical source absent

The current repository does not contain the WooTourism implementation. Code remediation is blocked until the canonical production source is identified and made auditable.

This is a source-availability blocker, not an architectural PENDING.

### PENDING

None created by this documentation change.

If the future audit discovers a decision involving a new persistent cache, new source of truth, new endpoint, new background job or changed price semantics, that specific decision must be documented and approved before implementation.

## 11. Completion criteria

WooTourism's part of this remediation can only be considered complete when:

1. the canonical deployed source has been identified;
2. owner/source-of-truth and price contracts are verified;
3. profiling confirms or excludes WooTourism as a material contributor to the reported TTFB;
4. any necessary code changes are implemented in the canonical owner rather than duplicated in this placeholder repository;
5. the integration with `theleadpartner/all_ways_colombia` is revalidated.
