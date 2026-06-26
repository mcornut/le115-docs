# Spec — Pricing Engine

## Goal

Calculate a detailed quote for a selected stay.

## Inputs

- arrivalDate
- departureDate
- adults optional
- children optional

## Output

- number of nights
- nightly price list
- subtotal
- fees
- total
- applied rules
- validation errors if any

## Rules

- Each night uses the pricing period matching its date.
- Cleaning fee is added once per stay.
- Prices are not hardcoded.
- A quote can be calculated only if dates respect stay rules.

## Definition of Done

- Unit tests cover mixed pricing periods.
- Unit tests cover high season Saturday rule.
- Unit tests cover unavailable dates.
- Unit tests cover cleaning fee.
