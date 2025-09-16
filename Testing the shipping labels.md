## Testing Shipping Labels: Complete Walkthrough

Based on the current implementation, here's how to test the shipping functionality for both FedEx and Maersk:

### Prerequisites Setup

**Vendor Account:** rondusllc@gmail.com
**Buyer Account:** itoroukpe72@gmail.com

---

## **Phase 1: As a Buyer (itoroukpe72@gmail.com)**

### Step 1: Place a Test Order
1. **Login as buyer** at itoroukpe72@gmail.com
2. **Browse products** via `/buyer/portal`
3. **Add items to cart** and proceed to checkout
4. **Complete the order** - this will generate an order number like `ORD-20250821-XXXX`
5. **Note the order number** for tracking later

### Step 2: Track Your Order
1. **Visit** `/track-order` (UnifiedOrderTracking page)
2. **Enter your order number** and **email** (itoroukpe72@gmail.com)
3. **Track the order status** - you'll see the progression from:
   - Pending → Confirmed → Processing → Shipped → Delivered

---

## **Phase 2: As a Vendor (rondusllc@gmail.com)**

### Step 1: Access Vendor Dashboard
1. **Login as vendor** at rondusllc@gmail.com
2. **Navigate to** `/vendor/dashboard`
3. **Go to Order Management** section

### Step 2: Process the Order
1. **Find the order** placed by itoroukpe72@gmail.com
2. **Mark as Processing** (if not already)
3. **Choose your shipping carrier** - you'll see these options:

#### For FedEx Labels:
```
Generate FedEx Label button
```
- Creates shipping label via `create-fedex-shipment` function
- Updates order status to 'shipped'
- Adds tracking number automatically

#### For Maersk Booking:
```
Generate Maersk Booking button  
```
- Creates sea freight booking via `create-maersk-shipment` function
- Generates container booking reference
- Provides port information and estimated arrival

---

## **Phase 3: Testing Interface (Admin/Development)**

### Step 1: Access Logistics Dashboard
1. **Navigate to** `/logistics/dashboard` (admin access needed)
2. **Use dedicated test interfaces:**

#### FedEx Testing:
- **FedEx Test Interface** tab
- Test rate calculations
- Test label generation with sample data

#### Maersk Testing:
- **Maersk Test Interface** tab
- Test sea freight rates
- Test booking creation

---

## **Current Implementation Status & Expected Behavior**

### ✅ **What Works Now:**

1. **FedEx Integration:**
   - Rate calculation
   - Label generation
   - Tracking integration
   - Automatic order status updates

2. **Maersk Integration:**
   - Booking creation
   - Container assignment
   - Port-to-port routing
   - Estimated delivery dates

3. **Order Workflow:**
   - Automatic carrier selection based on weight/destination
   - Order status progression
   - Email notifications

### 🔄 **Expected Testing Flow:**

**For FedEx (Air/Ground):**
```
Order Placed → Processing → Generate FedEx Label → 
Tracking Number Created → Status: Shipped → Delivery Updates
```

**For Maersk (Sea Freight):**
```
Order Placed → Processing → Generate Maersk Booking → 
Container Assigned → Port Loading → Transit → Port Discharge → Delivered
```

---

## **Detailed Testing Steps**

### **Scenario 1: Light Package (FedEx)**
1. **As Buyer:** Order lightweight items (< 5kg)
2. **As Vendor:** Generate FedEx label
3. **Verify:** Label PDF download, tracking number creation
4. **Track:** Use generated tracking number to monitor delivery

### **Scenario 2: Heavy/Bulk Order (Maersk)**
1. **As Buyer:** Order heavy/bulk items (> 20kg) 
2. **As Vendor:** Generate Maersk booking
3. **Verify:** Container booking reference, port details
4. **Track:** Monitor container movement through ports

---

## **Key Testing Points to Verify:**

### **Vendor Side:**
- [ ] Order visibility in vendor dashboard
- [ ] Carrier selection buttons appear
- [ ] Label/booking generation works
- [ ] Order status updates automatically
- [ ] Tracking numbers are stored

### **Buyer Side:**
- [ ] Order tracking works with order number + email
- [ ] Status progression is visible
- [ ] Shipping information displays correctly
- [ ] Tracking numbers are accessible

### **System Integration:**
- [ ] Email notifications are sent
- [ ] Database updates occur correctly
- [ ] API integrations function properly
- [ ] Error handling works as expected

This comprehensive testing approach will validate both FedEx and Maersk shipping label functionality from both vendor and buyer perspectives using your specified test accounts.
