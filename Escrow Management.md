## What Happens When Escrow Release Buttons Are Clicked

Based on the code analysis, here's what happens with each button in the Escrow Management system:

### 🚢 **"Release Shipping" Button** (Stage 1 - Blue Button)
**When clicked:**

1. **Validates the order:**
   - Checks if shipping escrow is already released (returns error if yes)
   - Verifies order payment status is "paid"
   - Confirms order has been shipped OR has tracking number

2. **Updates the order record:**
   - Sets `shipping_escrow_status` to `'released'`
   - Records `shipping_released_at` timestamp
   - Updates `updated_at` timestamp

3. **Creates vendor payouts** (depending on shipping preference):
   - **If AfriXport handles shipping:** Logs that platform receives the shipping payment (no vendor payout created)
   - **If vendor handles own shipping:** Creates `vendor_payouts` record with:
     - `payout_type: 'shipping'`
     - `status: 'ready'`
     - `amount: order.shipping_amount`
     - Note indicating shipping escrow released with tracking

4. **Sends notifications:**
   - **Vendor notification:** Informs them shipping payment is released (or that AfriXport handles it)
   - **Customer notification:** Informs them order has been shipped with tracking info

5. **UI updates:**
   - Shows success toast: "Shipping Escrow Released"
   - Refreshes the escrow data table
   - Button becomes disabled/hidden as status changes to "released"

---

### 📦 **"Release Product" Button** (Stage 2 - Green Button)
**When clicked:**

1. **Validates the order:**
   - Checks if product escrow status is `'held'`
   - Verifies payment status is `'paid'`
   - Can check for delivery confirmation, buyer confirmation, or auto-release date

2. **Updates the order record:**
   - Sets `status` to `'delivered'`
   - Sets `escrow_status` to `'released'`
   - Records `delivered_at` timestamp
   - Stores `escrow_release_reason` (e.g., 'time_based', 'buyer_confirmed', 'delivery_confirmed')

3. **Calculates and creates/updates vendor payouts:**
   - Calls `calculate_vendor_payout()` RPC function to determine proper amounts
   - Calculates commission (default 8%), international fees, and taxes
   - **If payout exists:** Updates existing record with:
     - `status: 'ready'`
     - `payout_type: 'product'`
     - Calculated `amount`, `commission_amount`, `tax_amount`, and `net_amount_after_tax`
   - **If no payout:** Creates new `vendor_payouts` record with all calculated values

4. **Sends notification:**
   - **Customer notification:** Informs them order is delivered and product payment released to vendor
   - Mentions that shipping payment was released earlier when order shipped

5. **UI updates:**
   - Shows success toast: "Product Escrow Released" with message "Product payment released and order marked delivered"
   - Refreshes escrow data table
   - Button becomes disabled/hidden as status changes to "released"

---

### 🔄 **Key Differences Between the Two:**

| Aspect | Release Shipping (Stage 1) | Release Product (Stage 2) |
|--------|---------------------------|--------------------------|
| **Trigger** | When order ships | When order is delivered/confirmed |
| **Amount Released** | Shipping costs only | Product price (minus shipping & fees) |
| **Order Status** | Stays same (e.g., "processing") | Changes to "delivered" |
| **Escrow Field** | `shipping_escrow_status` → 'released' | `escrow_status` → 'released' |
| **Payout Calculation** | Simple (flat shipping amount) | Complex (commission, tax, fees deducted) |
| **Purpose** | Pay for shipping carrier costs | Pay vendor for product after successful delivery |

---

### ⚠️ **Important Notes:**

1. **Two-Stage Escrow System:**
   - The system uses a **two-stage escrow** to protect all parties
   - Stage 1 releases shipping funds when order ships (vendor can pay carrier)
   - Stage 2 releases product funds when delivery is confirmed (vendor gets paid for goods)

2. **Both buttons check:**
   - Admin permissions (via the edge functions)
   - Order payment status
   - Current escrow status (to prevent duplicate releases)

3. **The product release has additional logic for:**
   - Buyer protection period (7 days after delivery confirmation)
   - Dispute resolution status
   - Automatic time-based release dates
   - Buyer-confirmed delivery

Would you like me to create any modifications to this escrow release flow, such as adding confirmation dialogs, changing the validation rules, or adjusting the payout calculations?
