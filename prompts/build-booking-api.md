# Prompt — Build Booking API

Read:
- docs/03-Business-Rules.md
- docs/05-Data-Model.md
- docs/06-API.md
- specs/booking.md

Goal:
Implement public quote and stay request APIs.

Definition of Done:
- POST /quote returns detailed quote.
- POST /stay-requests creates request with quote snapshot.
- Invalid dates return clear errors.
- Pending request does not block dates.
