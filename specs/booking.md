# Spec — Booking / Stay Request

## Goal

Allow a traveler to create a stay request after receiving a detailed quote.

## Flow

1. Select arrival date.
2. Select departure date.
3. Optionally enter adults and children.
4. Display quote.
5. Fill contact fields.
6. Submit request.
7. Show confirmation.

## Required fields

- First name
- Last name
- Email
- Phone

## Optional fields

- Adults
- Children
- Message

## Business rule

A stay request does not block dates.

## Definition of Done

- Invalid dates are rejected.
- High season rules are enforced.
- Quote is displayed before submission.
- Quote snapshot is stored with request.
- Admin receives notification.
