# Esplenin Hotel — Reservation & Payment System

Source: Esplenin Hotel Project Documentation v0.4 (2026-11-24). The full document wins over this summary if they disagree.

## Overview

Esplenin is a web application for a hotel company with several branches in different countries. Guests search, book and pay online without an account, and manage their booking from a private link sent by email. Staff use a role-based portal to approve bookings, assign rooms, check guests in and out, take payments, manage rooms and prices, and read reports. It started as a school project (a C program) and is being rebuilt as a full-stack, production-style system. Solo project, owner: Monty.

## Goals

1. Guests can find, book, pay for and cancel a room by themselves
2. Each branch's staff can assign rooms, change bookings, check guests in and out, record payments and report
3. A room type is never sold more times than it has rooms
4. Complete history: reservations are cancelled, never deleted, and important actions are logged
5. Several branches, each with its own currency, timezone and tax rules
6. Practice a production-like setup: security, caching, rate limiting, monitoring, tests

## Core User Flow

1. Guest picks a branch, dates, guests and theme, then searches rooms
2. Guest opens a room type, sees the price breakdown, and books
3. Guest enters main guest details and chooses: full payment, down payment, or pay after checkout (where offered)
4. Card payment through Stripe, or a 6-digit email code for pay after checkout
5. Reservation is created as Pending and a confirmation email with a private link is sent
6. A receptionist assigns a matching room: reservation becomes Approved, room becomes Reserved
7. ID verified, then check-in (room Occupied), stay with charges, check-out (room Cleaning, then Available)
8. Guest receives a review request

## Roles

Guest (no account), Receptionist (`RECEPTIONIST`), Supervisor (`SUPERVISOR`), Branch Manager (`MANAGER`), Super Admin (`SUPER_ADMIN`). Each staff role gets one page; higher roles include the lower roles' sections.

## Features

### Guest
- Room search, price breakdown, booking, card payment, cancellation with automatic full refund
- Private manage link, booking history by secure email link, PDF invoice, reviews, privacy requests

### Front desk (Receptionist, Supervisor)
- Pending queue, room assignment, check-in/out, folio and charges, payments recorded outside Stripe
- Reschedule (max 2 date changes for a Receptionist), room board, no-show handling
- Supervisor: goodwill refunds up to the branch limit, void charges, room blocks, daily summary

### Management (Branch Manager, Super Admin)
- Staff invitations, rooms and pricing, tax rules, branch settings, outstanding balances, disputes, review hiding
- Branch and company dashboards, PDF reports, branches, global settings, master lists, legal versions, data requests, audit log

## Scope

### In Scope
- The 62 numbered scope items in S02 of the full document
- Languages: English, French, Japanese, Chinese. Prices in each branch's currency (no conversion)
- Card payments via Stripe (test mode); cash, bank transfer and external card recorded by staff

### Out of Scope
- More than one room per booking; seasonal, date-based or weekend pricing; promo codes and discounts
- Names of additional guests; guest accounts with passwords; branch maps or descriptions
- Online payment methods other than card; third-party booking sites; loyalty program; native mobile app
- Excel exports (PDF only); cancellation, late-cancellation or no-show fees; payment hold timer or Expired status
- Housekeeping role; raw card data anywhere in the system

## Success Criteria

1. A guest completes a booking without help in under 5 minutes
2. Load test: 50 guests book the last room at once, exactly one succeeds and the others are refunded
3. Every staff action is blocked outside the person's role and branch
4. The full flow works in Stripe test mode: pay, refund, dispute
5. Every numbered scope item has at least one automated test

## Priority

- MUST: guest booking and cancellation (Scope 1 to 17), room assignment, check-in/out, payments, automatic refunds, roles and branch rules, staff invitations, overbooking protection
- SHOULD: date changes, no-show and Pending expiry jobs, outstanding balances, reports, translations
- COULD: reviews, dark mode, analytics, data export and anonymization tooling
