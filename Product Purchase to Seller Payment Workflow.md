
# Product Purchase to Seller Payment Workflow – Implementation Plan

This document outlines the **end-to-end workflow** for processing a product purchase, from customer payment to final payout to the seller. The system leverages **escrow protection** to ensure buyer safety and vendor trust.

---

## Phase 1: Customer Purchase & Payment

**Objective:** Capture order details and initiate payment flow.
**Implementation:**

1. **Shopping Cart & Checkout**

   * Implement `CartContext` for cart management.
   * Use `CheckoutFlow` component for checkout steps.
   * Integrate promotion codes with automatic discount calculation.
   * Provide payment method options: **Stripe (instant)** or **Bank Transfer (ACH/SEPA/Wire)**.

2. **Payment Processing**

   * **Stripe Path**:

     * Create `create-payment` serverless function to generate Stripe checkout sessions.
     * Redirect customer to Stripe for secure payment.
   * **Bank Transfer Path**:

     * Create `create-bank-transfer` function with ACH/SEPA/Wire support.
   * Record initial order in `orders` table:

     * `status = pending`
     * `escrow_status = held`
   * Save product details in `order_items` table.

---

## Phase 2: Payment Verification & Escrow

**Objective:** Verify payment and hold funds securely in escrow.
**Implementation:**

1. **Payment Verification**

   * After checkout, redirect to `OrderSuccess` page.
   * Implement `verify-payment` function:

     * Validate Stripe session or confirm bank transfer.
     * Update `orders.status = confirmed`, `payment_status = paid`.
     * Retain funds in **escrow**.

2. **Commission & Payout Setup**

   * Insert commission record in `commissions` table (8% default rate).
   * Create vendor payout record in `vendor_payouts` table (92% of sale price).
   * Set payout status = `pending`.
   * Send confirmation email to customer.

---

## Phase 3: Order Processing & Fulfillment

**Objective:** Vendor fulfills the order.
**Implementation:**

1. Vendor notified of new order.
2. Vendor updates shipping details & tracking numbers.
3. Order status transitions: `confirmed → processing → shipped`.

---

## Phase 4: Delivery & Escrow Release

**Objective:** Confirm delivery and release funds from escrow.
**Implementation:**

1. **Delivery Confirmation**

   * Customer marks order as received.
   * Admin reviews delivery via `EscrowTools` component.
   * Admin triggers “Mark Delivered + Release Escrow” for order ID.

2. **Escrow Release**

   * Run `release-vendor-payout` function.
   * Update:

     * `orders.status = delivered`
     * `escrow_status = released`
     * `vendor_payouts.status = ready`.

---

## Phase 5: Final Payout Processing

**Objective:** Transfer funds to vendor’s bank account.
**Implementation:**

1. Process payouts with `status = ready`.
2. Initiate transfer to vendor’s bank account (bank details stored in vendor profile).
3. Update payout records with transfer IDs and completion timestamps.
4. Mark commission records as finalized.

---

## Security & Compliance Features

* **Escrow Protection**: Funds held until delivery confirmation.
* **Admin Controls**: Manual approval prevents premature payouts.
* **Audit Trail**: Full transaction history recorded in database.
* **Global Payment Support**: Stripe and Bank Transfer.
* **Commission Tracking**: Automated platform fee calculation.
* **Promotion Integration**: Discounts applied seamlessly across workflow.

---

## Timeline

* **Instant**: Payment initiation & escrow hold.
* **1–7 days**: Vendor processing & shipping.
* **Manual**: Escrow release upon delivery confirmation.
* **1–3 days**: Final vendor bank transfer.

---

## Key Insight

The money flow follows **three secure stages**:
**Customer Payment → Escrow Hold → Seller Release**.
This ensures **buyer protection** while providing vendors with **predictable payouts** once obligations are met.

---

