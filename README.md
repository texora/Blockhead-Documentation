# **Blockhead Payment Processor Specifications**

## **Overview**

Blockhead is an open-source, permissionless, reputation-based payment processor built on Polygon. Payments are processed in MATIC, utilizing secure escrow mechanisms, decentralized oracle integrations, and no-trust smart contracts to manage disputes and automate settlements.

Key Objectives:
- Enable secure payments with minimal trust requirements.
- Provide a scalable, decentralized framework for crypto invoicing.
- Automate dispute resolution through Marketplace Oracles.

### **Key Components**
1. **Website (`blockhead.box`):**  
   IPFS-hosted React app for invoice creation, payment management, and dispute handling.
   
2. **Smart Contracts:**  
   Solidity-based contracts to manage invoice lifecycle, escrow operations, and integration with oracles.

3. **Escrow Wallet:**  
   A deterministic, secure wallet for temporarily holding funds until release conditions are met.

4. **Oracles:**  
   - **Marketplace Oracles:** Manage invoices, disputes, and hold periods.
   - **Blockchain Oracles:** Monitor payment states and trigger smart contract actions.

---

## **Project Development Phases**

### **Phase 0: Specification and Platform Setup**
**Objectives:**
- Finalize specifications.
- Establish IPFS-hosted landing page (`blockheadverifybytoken.eth`).
- Enable wallet-based authentication.

**Deliverables:**
- Wireframe for IPFS landing page.
- Functional wallet login integrated with Polygon network.

**Budget:**  
- $1,500 + bonus.

---

### **Phase 1: Basic Features and Smart Contracts**
**Objectives:**
- Implement foundational functionality for admins, creators, and payers.
- Deploy basic smart contracts to manage invoice creation, payment, and state transitions.
- Conduct initial audit to verify smart contract security.

**Deliverables:**
1. **Features:**  
   - **Invoice Management:** Creation, acceptance, and cancellation.  
   - **Payment Handling:** Monitor payments and manage state changes.  
   - **Admin Controls:** Configure key settings like fees and escrow wallet.

2. **Documentation:**  
   Comprehensive guide to smart contract methods, API endpoints, and frontend features.

3. **Security Audit:**  
   Pass Audit 1 with third-party verification.

**Budget Breakdown:**  
- Implementation: $5,000  
- Documentation: $1,000  
- Audit 1: $3,000  

**Total:** $9,000 + bonus.

---

### **Phase 2: Advanced Features and Integrations**
**Objectives:**
- Introduce advanced user workflows and refund mechanisms.
- Enable Marketplace Oracle integrations for disputes and custom hold periods.
- Support fiat conversion and external payment gateways.

**Deliverables:**
1. **Features:**  
   - Refund management for underpayments, overpayments, and cancellations.  
   - Marketplace Oracle dispute resolutions and custom hold periods.  
   - Fiat conversion support using third-party APIs.

2. **Security Audit:**  
   Pass Audit 2 to ensure new functionality is secure.

**Budget Breakdown:**  
- Implementation: $5,000  
- Documentation: $1,000  
- Audit 2: $3,000  

**Total:** $9,000 + bonus.

---

### **Phase 3: Enhanced Features and Scalability**
**Objectives:**
- Add advanced admin features (multi-signature approvals, zk-rollups).
- Enhance user experience with notifications and streamlined UI.
- Ensure scalability and security through rigorous testing.

**Deliverables:**
1. **Features:**  
   - zk-rollups for efficient transaction processing.  
   - Multi-signature admin confirmations for critical actions.  
   - Notification system for transaction updates.  

2. **Security Audit:**  
   Pass Audit 3 to validate scalability and final feature set.

**Budget:**  
$8,000 + bonus.

---

## **Functional Specifications**

### **1. Admin Functions**
Accessible via `/admin`, admin functions are controlled through secure key authentication.

#### **Features and Methods**
1. **Admin Authentication:**  
   Securely authenticate admins using wallet keys stored in the smart contract.  
   ```solidity
   mapping(address => bool) public admins;

   function isAdmin(address _address) public view returns (bool) {
       return admins[_address];
   }

   function addAdmin(address _newAdmin) public onlyAdmin {
       admins[_newAdmin] = true;
   }
   ```

2. **Transfer Admin Control:**  
   Allow admin role transfer with secondary confirmation.  
   ```solidity
   function transferAdmin(address _newAdmin) external onlyAdmin {
       require(_newAdmin != address(0), "Invalid admin address");
       pendingAdmin = _newAdmin;
   }

   function confirmTransfer() external {
       require(msg.sender == pendingAdmin, "Not pending admin");
       admin = pendingAdmin;
   }
   ```

3. **Shut Off Switch:**  
   Pause all contract operations during emergencies.  
   ```solidity
   function emergencyPause() external onlyAdmin {
       _pause();
   }

   function resume() external onlyAdmin {
       _unpause();
   }
   ```

4. **Set Fees and Configurations:**  
   Configure gas fees, escrow wallets, and marketplace oracle addresses.  

---

### **2. Invoice Management**
Invoice creation, payment, and lifecycle management are the core features of Blockhead.

#### **Invoice Creation Workflow**
1. Creator enters details (amount, expiration, payer address).
2. Smart contract validates input and stores data securely.
3. System generates a unique invoice ID, escrow address, and payment link.

#### **Smart Contract Example**
```solidity
struct Invoice {
    uint256 id;
    address creator;
    address payer;
    uint256 amount;
    uint256 expiration;
    string state; // "Created", "Paid", "Accepted", etc.
}

Invoice[] public invoices;

function createInvoice(
    address _payer,
    uint256 _amount,
    uint256 _expiration
) external returns (uint256) {
    require(_expiration > block.timestamp, "Expiration must be in the future");

    uint256 invoiceId = invoices.length;
    invoices.push(Invoice({
        id: invoiceId,
        creator: msg.sender,
        payer: _payer,
        amount: _amount,
        expiration: _expiration,
        state: "Created"
    }));

    return invoiceId;
}
```

---

### **3. Payment Handling**
Payers interact with invoices via unique payment URLs. Blockchain Oracles monitor payments and update states.

#### **States:**
- `Created`: Invoice created and awaiting payment.
- `Paid`: Payment received; awaiting creator acceptance.
- `Underpaid/Overpaid`: Payment mismatch with invoice amount.
- `Cancelled`: Invoice cancelled; funds refunded.
- `Disputed`: Escrow funds held for Marketplace Oracle resolution.

---

### **4. Refunds**
Refunds are triggered automatically for:
1. Underpayments exceeding gas fees.
2. Cancelled invoices.
3. Marketplace Oracle reallocations.

#### **Refund Workflow**
1. Funds returned to payer’s sending address.  
2. State updated to "Refunded".

#### **Smart Contract Example**
```solidity
function refund(uint256 _invoiceId) external {
    Invoice storage invoice = invoices[_invoiceId];
    require(invoice.state == "Cancelled", "Invoice not eligible for refund");

    payable(invoice.payer).transfer(invoice.amount);
    invoice.state = "Refunded";
}
```

---

### **5. Marketplace Oracles**
Marketplace Oracles enhance functionality by enabling:
1. **Custom Hold Periods:** Assign specific hold durations per invoice creator.  
2. **Dispute Resolution:** Reallocate funds between payer and creator.

#### **Integration Example**
Utilize Chainlink to automate invoice state updates and release funds:  
```solidity
function settleDispute(uint256 _invoiceId, uint256 _creatorAmount, uint256 _payerAmount) external onlyOracle {
    require(_creatorAmount + _payerAmount == invoices[_invoiceId].amount, "Invalid allocation");
    
    payable(invoices[_invoiceId].creator).transfer(_creatorAmount);
    payable(invoices[_invoiceId].payer).transfer(_payerAmount);
    invoices[_invoiceId].state = "DisputedSettled";
}
```

---

Here’s the refined, enriched, and organized specification for the **Website**, **Escrow Wallet**, and **Notifications** sections. These improvements include precise details, enhanced structure, and additional context. This will be integrated into the overall document seamlessly.

---

## **Website: `blockhead.box`**

The Blockhead website is a React-based static site hosted on IPFS, providing the frontend interface for users to manage invoices and payments. 

### **Hosting and Infrastructure**
- **Hosting:**  
  - IPFS via Fleek or 4everland for decentralized, robust hosting.
  - Test server deployment on Vercel during development and testing phases.  
- **Domain Management:**  
  - `.box` domain handled via `my.box` and `3dns.box`.

### **Core Features and Pages**
The website uses **WalletConnect** for user authentication and features a responsive, Bootstrap-based design with a right-sidebar layout. Minimal styling is applied initially, with advanced designs planned for **Phase 3**.

#### **1. Default Landing Page**
- **Purpose:** Serve as the entry point for Blockhead users.  
- **Contents:**  
  - **Explanation Text:** Brief overview of Blockhead and its features.  
  - **Login:** WalletConnect login button for users.  
  - **GitHub Link:** Redirect to the project’s GitHub repository for transparency.  

#### **2. Navbar**
- **Dynamic Links:**  
  - `Create Invoice`  
  - `Invoice List`  
  - `Find Invoice`  
  - `Admin` (visible only to wallets with admin privileges).  

#### **3. Pay Invoice Page**
- **Purpose:** Allow payers to view invoice details and make payments.  
- **Access:** URL is shared by the Invoice Creator with the Invoice Payer.  
- **Features:**  
  - Displays all invoice details except the payment address.  
  - Requires WalletConnect login to reveal the payment address.

#### **4. Create Invoice Page**
- **Purpose:** Enable Invoice Creators to generate new invoices.  
- **Features:**  
  - Fields to enter invoice details (amount, expiration, payer wallet address).  
  - Generates a unique Invoice ID and corresponding payment link.  

#### **5. Invoice List Page**
- **Purpose:** Allow logged-in users to view a list of invoices.  
- **Features:**  
  - Sortable columns: status, date created, date paid, creator ID, payer ID.  
  - Search functionality by Invoice ID, address, or state.  

#### **6. Invoice Page**
- **Purpose:** Display the current state and history of an individual invoice.  
- **Features:**  
  - Details: Invoice ID, amount, status, payer, and creator.  
  - Actions: Creators can cancel the invoice directly from this page.  

#### **7. Admin Page**
- **Purpose:** Provide administrators with controls to configure system settings.  
- **Access:** Restricted to wallets with admin privileges.  
- **Features:**  
  - Adjust fees, configure oracles, and manage escrow wallet addresses.  
  - Set blacklists, hold times, and trigger circuit breakers.

---

## **Escrow Wallet**

The **Escrow Wallet** is a critical component of the Blockhead system, securely holding funds until release conditions are met.

### **Key Features**
1. **Deterministic Address Generation:**  
   Each invoice payment receives a unique wallet address derived deterministically from a secure key set by the admin.  

2. **Payment Security:**  
   - Funds are locked in the escrow wallet until the invoice is accepted, refunded, or disputed.  
   - Supports refunds for overpayments or cancellations.  

3. **Admin Configuration:**  
   The wallet address is configured and maintained by the admin, ensuring flexibility and control.

### **Planned Enhancements (Phase 2):**
- Advanced security mechanisms for deterministic address generation.  
- Integration with zk-rollups for scalability and reduced transaction fees.  

---

## **Notifications**

### **Phase 2: Marketplace Oracle Notifications**
- Notifications generated through the **Marketplace Oracle** to inform users of key state changes.  
- Examples of notifications:  
  - Dispute initiated by a Marketplace Oracle.  
  - Resolution or allocation of escrow funds.

### **Phase 3: Wallet Messaging**
- **Real-Time Updates:**  
  Use wallet-based messaging protocols to notify users about the status of their invoices, payments, or disputes.  
- **Examples:**  
  - Invoice accepted or cancelled.  
  - Payment received or refunded.  

---

## **Summary of Enhancements**

1. **Website:**  
   - Organized feature list for each page, emphasizing user roles and dynamic functionality.  
   - Clarity on hosting and domain infrastructure.  

2. **Escrow Wallet:**  
   - Added detailed information on deterministic address generation and admin configurations.  
   - Highlighted future scalability with zk-rollups.  

3. **Notifications:**  
   - Clear separation of notification phases (Marketplace Oracle vs. Wallet Messaging).  
   - Examples of real-time updates to enhance user experience.  

This refined and enriched specification will enhance usability and technical clarity while integrating seamlessly into the overarching Blockhead project framework.
