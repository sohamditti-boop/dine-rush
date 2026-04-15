# Dine Rush POS - Client Handoff & Setup Guide

## 1. Initial Data Setup Guide (For Owners)
Before your restaurant opens, you must populate the system in this exact order:

1. **Inventory Setup:** Go to *Inventory*. Add all raw materials (e.g., Rice, Chicken, Tomatoes). **CRITICAL:** Assign an accurate "Cost/Unit" to every item. Without this, your Food Cost Reports will be ₹0.
2. **Menu Setup:** Go to *Menu*. Create the dishes your customers buy (e.g., Chicken Curry - ₹250).
3. **Recipe Linking:** Inside *Menu*, click the Recipe link for each dish. Map the exact raw materials consumed by that dish (e.g., 1 order of Chicken Curry = 0.25kg Chicken + 0.1kg Tomatoes). This enables the automatic inventory deduction.
4. **Table Setup:** Go to *Tables*. Add your physical restaurant table layout.
5. **Staff Accounts:** Go to *Staff*. Create logins for Managers and Waiters. Assign the correct permissions.

---

## 2. Staff Training Guide (For Waiters & Managers)

### How to Bill a Table
1. Go to **Billing** using your tablet or PC.
2. Select the items the customer wants to order to add them to the cart.
3. Enter the Table Number.
4. Click **Send to Kitchen & Save**.
   - This creates a KOT (Kitchen Order Ticket).
   - *Note: If raw materials are out of stock, the system will absolutely block the KOT. A manager must restock the inventory from the system.*
5. When the customer is ready to pay, go to **Orders**.
6. Find the table under "Running Orders". Click **Checkout**.
7. Apply any discounts, select the Payment Mode, and click **Settle & Print Bill**.

### How to Restock Inventory
1. Go to **Inventory**.
2. Find the item you bought from the market.
3. Click the **+ (Restock)** button.
4. Enter the quantity added and the supplier name.
   - *This will automatically increase stock and log a purchase record.*

### How to Read Reports
1. **Managers:** Go to **Reports** to see daily Sales, Food Costs, and Gross Profit overviews.
2. **Owners:** Go to **Financials** to generate end-of-month holistic P&L reports (counting total sales minus material purchases minus expenses).

---

## 3. Firebase Security Rules
*To be configured securely by the Developer before launch.*
Go to Firebase Console -> Firestore Database -> Rules. Replace the existing rules with these:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Auth Check Helpers
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function getUserRole() {
      return get(/databases/$(database)/documents/user_roles/$(request.auth.uid)).data.role;
    }
    
    function isSuperAdmin() {
      return isAuthenticated() && getUserRole() == 'super_admin';
    }
    
    function isManagerOrAdmin() {
      return isAuthenticated() && (getUserRole() == 'manager' || getUserRole() == 'super_admin');
    }
    
    function isStaffOrHigher() {
      return isAuthenticated() && (getUserRole() == 'staff' || getUserRole() == 'manager' || getUserRole() == 'super_admin');
    }

    // Rules
    match /user_roles/{userId} {
      allow read: if isAuthenticated() && request.auth.uid == userId;
      allow write: if isSuperAdmin(); // Only admin can assign roles
    }

    match /users/{userId} {
      allow read: if isStaffOrHigher();
      allow write: if isSuperAdmin();
    }

    match /menu/{document=**} {
      allow read: if isStaffOrHigher();
      allow write: if isManagerOrAdmin();
    }

    match /inventory/{document=**} {
      allow read: if isStaffOrHigher();
      allow write: if isManagerOrAdmin();
    }
    
    match /recipes/{document=**} {
      allow read: if isStaffOrHigher();
      allow write: if isManagerOrAdmin();
    }

    match /orders/{document=**} {
      allow read, write: if isStaffOrHigher();
    }

    match /kots/{document=**} {
      allow read, write: if isStaffOrHigher();
    }

    match /tables/{document=**} {
      allow read, write: if isStaffOrHigher();
    }

    match /expenses/{document=**} {
      allow read, write: if isManagerOrAdmin();
    }

    match /purchases/{document=**} {
      allow read, write: if isManagerOrAdmin();
    }
    
    match /inventory_transactions/{document=**} {
      allow read, write: if isStaffOrHigher();
    }

    // Default Deny
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```
