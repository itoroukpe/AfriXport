## Commission Structure

**Category-Based Commission Rates:**
- **Arts & Crafts**: 10.0%
- **Electronics & Tools**: 6.0%
- **Fashion & Accessories**: 12.0%
- **Home Décor & Furniture**: 10.0%
- **Processed Food & Beverages**: 8.0%
- **Wholesale B2B**: 4.5%

**Additional Fees:**
- **International Processing Fee**: 4% (applied to all orders)

## How Commission is Calculated and Applied

### 1. **Order Payment Processing**
When a customer pays for an order (via `verify-payment` edge function):

1. **Commission Record Creation**: For each item in the order:
   - Base commission is calculated using the category-specific rate
   - International processing fee (4%) is added
   - Total commission = (Base Commission Rate × Order Amount) + (4% × Order Amount)
   - A record is created in the `commissions` table with status 'pending'

2. **Vendor Payout Calculation**: Using the `calculate_vendor_payout` database function:
   - Gross amount = Order total
   - Commission deducted = Category rate + 4% international fee
   - Tax calculated on remaining amount (15% VAT for South Africa, varies by jurisdiction)
   - Net payout = Gross - Commission - International Fee - Tax

### 2. **Escrow System**
- Vendor payouts are created with status 'pending' and held in escrow
- Funds are automatically released after order delivery confirmation
- This protects both buyers and sellers

### 3. **Special Incentives Available**
Based on the `useCommissionRates` hook:

- **First 3 Months Bonus**: 0% commission (only 4% international fee applies)
- **Volume Discount**: 2% commission reduction for sellers doing >$5,000/month
- **Minimum Rate**: Total fees never go below 2% (including international fee)

### 4. **Payout Flow**
1. **Order Placed & Paid**: Commission calculated and vendor payout created (status: 'pending')
2. **Order Delivered**: Status can change to 'ready' for payout
3. **Admin Processing**: Admin can release payouts through the admin dashboard
4. **Payment**: Vendor receives net amount after all deductions

### 5. **Example Calculation**
For a $100 Fashion & Accessories order:
- Base Commission: $100 × 12% = $12
- International Fee: $100 × 4% = $4
- Subtotal after fees: $100 - $12 - $4 = $84
- Tax (15% VAT): $84 × 15% = $12.60
- **Vendor Receives**: $84 - $12.60 = **$71.40**

### 6. **Tracking & Transparency**
- Vendors can view all commission details in their dashboard
- Commission breakdown shows base rate, international fee, and taxes
- Real-time payout status tracking
- Historical commission data available

This system ensures fair, transparent commission handling while protecting all parties through the escrow mechanism and providing competitive rates based on product categories.
