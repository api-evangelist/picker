---
name: Create and track a Picker delivery
description: Quote, create, track, and (if needed) cancel a last-mile delivery booking with the Picker API, and subscribe to lifecycle webhooks.
api: openapi/picker-openapi-original.json
method: generated
generated: '2026-07-20'
operations:
  - postApiCheckregion
  - postBookingPrecheckout
  - postBookingValidatebooking
  - postBookingCreatebooking
  - getApiGetbookingdetails
  - getTrackingGetusertrackingdata
  - putBookingCancelbookingrequest
  - postApiWebhooks
---

# Create and track a Picker delivery

Picker is a Latin American last-mile delivery orchestration platform. This skill covers the merchant happy path: confirm coverage, quote, create a booking, track it, and handle cancellation and event webhooks.

## Authentication
- Send your per-store API Key in the `authorization` HTTP header on every request (obtained from the Picker Dashboard, Account/Locales; requires the Pro Plan).
- Production host: `https://api.pickerexpress.com`. Test host: `https://dev-api.pickerexpress.com`.
- Optionally set `content-language` (ES/EN) and `utcoffset` headers.

## Steps

1. **Confirm the region is serviced.** Call `postApiCheckregion` (`POST /api/checkRegion`) with the pickup/drop-off location to get the corresponding region data. If none is returned, the address is out of coverage — stop.

2. **Get a quote.** Call `postBookingPrecheckout` (`POST /booking/preCheckout`) to compute the delivery fee and details for the order before committing.

3. **Validate.** Call `postBookingValidatebooking` (`POST /booking/validateBooking`) to check the booking can be created before creating it.

4. **Create the booking.** Call `postBookingCreatebooking` (`POST /booking/createBooking`). Persist the returned booking id.

5. **Subscribe to lifecycle webhooks (once, per store).** Call `postApiWebhooks` (`POST /api/webhooks`) to subscribe to events — notably **Driver Assigned** and **Update Booking Status** — so you receive status changes instead of polling. List with `getApiWebhooks`, remove with `deleteApiWebhooks`.

6. **Track progress.** Call `getApiGetbookingdetails` (`GET /api/getBookingDetails`) for full booking state, and `getTrackingGetusertrackingdata` (`GET /tracking/getUserTrackingData`) for the live tracking path.

7. **Cancel if needed.** Call `putBookingCancelbookingrequest` (`PUT /booking/cancelBookingRequest`) to request cancellation. Cancellation charges may apply.

## Conventions & error handling
- Pagination on list endpoints uses `limit`/`skip` (and sometimes `page`) plus `sortField`/`sortOrder`. See `conventions/picker-conventions.yml`.
- There is **no** documented idempotency-key; avoid blind retries of `createBooking` — reconcile via `getBookingDetails` before retrying.
- The published spec declares only `default` responses (no typed error envelope); treat non-2xx bodies defensively.
