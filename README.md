# **Blockhead Documentation**

## **Overview**

**Blockhead** is an open-source, permissionless, and reputation-bound payment processor built on the **Polygon network**. It allows for secure, trustless payments between users, with payment dispute resolution handled by **Marketplace Oracles**. Blockhead is built for scalability, transparency, and decentralized trust, making it an ideal platform for handling crypto payments within decentralized marketplaces.

The platform uses **MATIC** as its base currency for transactions and incorporates a **reputation-based model** for users (via oracles) to ensure fair dispute resolution and effective payment processing.

---

## **Key System Components**

### **1. Website (blockhead.box)**  
The **blockhead.box** website is a **React**-based static page hosted on **IPFS**. It provides the following functionalities:

- **Invoice Management**: Users can create, view, accept, and cancel invoices.
- **User Login**: Wallet-based login via **WalletConnect**, allowing users to interact with the platform in a secure and decentralized manner.
- **Invoice Payment**: Invoice payers can view invoice details and make payments securely.
- **Admin Panel**: A secure admin interface to configure settings like the **escrow wallet**, **oracle addresses**, and **gas fees**.

### **2. Smart Contracts (Solidity)**  
The **smart contracts** are the backbone of Blockhead's decentralized payment processing. They handle:

- **Invoice Creation**: Smart contracts generate and store invoice data and ensure funds are held in escrow until all conditions are met.
- **Escrow Wallet Management**: Funds are held in a secure escrow wallet until they are released or refunded based on the invoice creator's and payer’s actions.
- **Dispute Resolution**: Marketplace Oracles trigger the reallocation of funds in case of disputes.
- **Payment State Transitions**: The smart contracts monitor payment status and update invoice states (e.g., **Paid**, **Accepted**, **Cancelled**, etc.).

### **3. Escrow Wallet**  
The **Escrow Wallet** is used to hold funds temporarily before they are released to the invoice creator or refunded to the payer. The wallet's address is generated deterministically using an admin-configured key, ensuring a secure, unique address for each transaction.

- **Escrow Address Generation**: Each invoice creates a unique wallet address based on the smart contract's logic.
- **Hold Period**: The smart contract determines the hold period, based on either the marketplace oracle’s configuration or the settings set by the Blockhead admin.
  
### **4. Marketplace Oracles**  
Marketplace Oracles play a crucial role in creating and managing invoices, as well as resolving disputes. Oracles in the Blockhead ecosystem provide additional functionalities, including:

- **Invoice Creation**: Marketplace oracles create invoices on behalf of sellers and include additional parameters, such as custom invoice terms and dispute resolution rules.
- **Hold Periods**: Oracles can set custom hold periods for invoices based on the marketplace's rules.
- **Dispute Resolution**: If an invoice is disputed, the oracle can trigger fund reallocations from the **Escrow Wallet** to either the creator or the payer.

**Example Oracle Network**: **Chainlink** (permissionless oracle network)

### **5. Blockchain Oracles**  
Blockchain Oracles monitor the blockchain for updates regarding payments. These oracles:

- **Monitor Payments**: They track transactions on the blockchain to confirm payments to the Escrow Wallet and update invoice statuses accordingly.
- **State Changes**: Based on payment detection, they trigger state transitions in the invoice (e.g., from **Created** to **Paid** or **Underpaid**).
  
---

## **Developer Payments**

### **$30,000 + Bonus**

#### **Phase 0: Specification and Platform**

- **Task**: Edit and complete the documentation, establish wireframes for the landing page, and implement wallet login functionality.
- **Payment**: $1,500 + bonus

#### **Phase 1: Basic Functions and Smart Contracts**

- Invoice management (create, accept, cancel)
- **Payer Functionality**: Invoice payment and release
- **Admin Functions**: Configure oracle, escrow, and set gas fees
- **Payment**: $5,000  
- **Documentation Completion**: $1,000  
- **Audit 1 Completion**: $3,000  
- **Total for Phase 1**: $9,000 + bonus

#### **Phase 2: User-Enabling Functions and Smart Contracts**

- Invoice list, search, and sorting
- Refund functions (overpay, cancel)
- Marketplace oracle orders and dispute allocations
- Gas fee configuration, fiat conversion
- **Payment**: $5,000  
- **Documentation Completion**: $1,000  
- **Audit 2 Completion**: $3,000  
- **Total for Phase 2**: $9,000 + bonus

#### **Phase 3: Additional Functions and Contracts**

- Admin transfer, second admin confirmation
- ZK Rollups, user-set terms
- Marketplace oracle hold periods, more site features
- **Payment**: Estimated $8,000 + bonus

---

## **System Architecture and Interactions**

### **1. Admin Functions**

**Blockhead Admin** has control over the system settings via a secure admin panel (accessed via **blockhead.box/admin**). Admin functions include:

- **Admin Login**: A secure login process using wallet-based authentication. The first admin key and confirmation key are encoded within the smart contract.
- **Transfer Admin**: Admin can transfer control using a second confirmation key for extra security.
- **Shut Off Switch**: Temporarily disables all operations in case of emergency or maintenance.
- **Configure Oracle**: Admin can configure the **Marketplace Oracle** address that governs invoice creation and dispute resolution.
- **Set Gas Fee**: Admin can configure a fixed gas fee, and adjust it in extreme network conditions (Phase 2 will automate this process based on real-time data).
- **Escrow Management**: Admin configures the **Escrow Wallet** address and sets the **Blockhead Fee Margin**.

---

### **2. Invoice Management**

**Invoice Creation**:

1. **Invoice Data Input**:
   - **Amount** (in MATIC)
   - **Expiration** (maximum of 180 days)
   - **Release Address** (where funds will be sent upon release)
   - **Public Key** (for added security)

2. **Smart Contract Integration**:
   The smart contract automatically adds:
   - **Escrow Wallet Address**
   - **Release Date**
   - **Invoice ID**
   - **Payment URL and QR code** for easy access.

3. **Blockchain Oracle Monitoring**:  
   The **Blockchain Oracle** monitors the blockchain to check for incoming payments. It continuously checks the blockchain until payment is received or the invoice expires.

---

### **3. Payment Process**

1. **Payment Detection**:  
   Once the payer makes a payment, the **Blockchain Oracle** detects it and updates the invoice status (e.g., **Paid**, **Underpaid**).
  
2. **Invoice Acceptance**:  
   The **Invoice Creator** reviews the payment. If the payment is underpaid, they can either accept the payment or cancel the invoice. If the payment is overpaid, the excess is refunded to the payer.

3. **Invoice Release**:  
   The invoice is released to the **Invoice Creator** once all conditions are met (e.g., invoice is accepted, hold period expired).

---

### **4. Marketplace Oracle Functionality**

Marketplace oracles are responsible for:

- **Invoice Creation**: Marketplace Oracles create invoices that include all relevant details, such as invoice terms, hold periods, and dispute resolution rules.
- **Dispute Resolution**: In the event of a dispute, oracles can adjust the invoice’s release timing and allocate funds accordingly.
- **Hold Period Configuration**: Oracles can set specific hold periods for individual invoices or rely on the default set by the admin.

---

### **5. Escrow Wallet Interaction**

Each invoice generates a unique escrow wallet address for the payment to be sent to. The **Escrow Wallet** holds funds until all conditions are met:

- **Invoice Acceptance**: Funds are released to the **Invoice Creator** once the invoice is accepted.
- **Refund Conditions**: If a payment is underpaid or the invoice is canceled, funds are refunded to the **Invoice Payer**.
- **Dispute-Triggered Releases**: If there is a dispute, the **Marketplace Oracle** can reallocate funds to either the creator or the payer.

---

### **6. Notifications and Messaging**

- **Marketplace Oracle Notifications**: Oracles provide updates on the status of disputes, invoice payments, and any changes to the payment state.
- **Wallet Messaging** (Phase 3): This feature will allow for direct notifications sent to users through their wallets to update them on invoice statuses, disputes, and releases.

