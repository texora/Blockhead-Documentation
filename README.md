## **Overview**

Blockhead is a decentralized, open-source, reputation-based payment processor built on Polygon. It facilitates crypto payments in MATIC using deterministic escrow wallets and decentralized oracles for trustless operations and dispute resolution.

### **Key Features**
1. **Secure Payments:**  
   No-trust, deterministic payments stored in smart contract-controlled escrow wallets.
   
2. **Reputation-Bound Dispute Resolution:**  
   Marketplace Oracles resolve disputes based on pre-configured rules, with transparency and fairness.

3. **Permissionless Functionality:**  
   Open to all users without restrictions, enabling invoice creation and payment management.

4. **Automated Refunds:**  
   For overpayments, underpayments, and canceled transactions.

### **Core Components**
1. **Website:**  
   IPFS-hosted React application for user interaction, including invoice creation and payment tracking.
   
2. **Smart Contracts:**  
   Solidity contracts handle payment processing, state management, refunds, and oracle integration.

3. **Escrow Wallet:**  
   Secure wallet holding funds temporarily until predefined release conditions are met.

4. **Marketplace Oracles:**  
   External entities validate invoices, manage escrow hold periods, and resolve disputes.

5. **Blockchain Oracles:**  
   Automated systems monitor on-chain activity and trigger contract state changes.

---

## **Development Phases and Budget**

### **Phase 0: Specification and Platform Setup**
**Objectives:**
- Finalize specifications.
- Develop an IPFS-hosted landing page (`blockheadverifybytoken.eth`).
- Enable wallet authentication using WalletConnect.

**Deliverables:**
- Landing page wireframe hosted on IPFS.
- Integrated Polygon-compatible wallet login.

**Budget:** $1,500 + bonus.

---

### **Phase 1: Basic Functions and Smart Contracts**
**Objectives:**
- Develop core features:
  - Admin controls.
  - Invoice creation, acceptance, and cancellation.
  - Payment processing and escrow management.
- Deploy foundational smart contracts.
- Pass initial security audit.

**Deliverables:**
- Functional website integrated with smart contracts.
- Comprehensive documentation.
- Audit 1 approval.

**Budget:** $9,000 + bonus.

---

### **Phase 2: User-Enhancing Features**
**Objectives:**
- Add refunds, dispute handling, fiat conversion, and Marketplace Oracle integration.
- Enable dynamic hold periods for invoices through Marketplace Oracles.

**Deliverables:**
- User-friendly interfaces and advanced workflows.
- Integration of Marketplace Oracles for dispute and hold period management.
- Audit 2 approval.

**Budget:** $9,000 + bonus.

---

### **Phase 3: Scalability and Advanced Features**
**Objectives:**
- Add zk-rollups for scalability.
- Introduce wallet notifications and enhanced admin controls.
- Enable advanced contract features like multi-signature approvals and transaction disposal.

**Deliverables:**
- Fully scalable system with zk-rollups and wallet messaging.
- Advanced features integrated into smart contracts.
- Audit 3 approval.

**Budget:** $8,000 + bonus.

---

## **Functional Specifications**

### **Admin Functions**

Admins control the system using `/admin`, with actions restricted to authorized wallet addresses.

---

#### **Admin Authentication**
**Description:** Admins authenticate using their wallet keys, which are encoded in the smart contract.

**Example Code:**
```solidity
mapping(address => bool) public isAdmin;

modifier onlyAdmin() {
    require(isAdmin[msg.sender], "Not authorized");
    _;
}

function addAdmin(address _admin) external onlyAdmin {
    isAdmin[_admin] = true;
}

function removeAdmin(address _admin) external onlyAdmin {
    isAdmin[_admin] = false;
}
```

---

#### **Transfer Admin Control**
**Description:** Allows the current admin to transfer control to another wallet, with secondary confirmation.

**Example Code:**
```solidity
address public pendingAdmin;

function initiateAdminTransfer(address _newAdmin) external onlyAdmin {
    require(_newAdmin != address(0), "Invalid address");
    pendingAdmin = _newAdmin;
}

function confirmAdminTransfer() external {
    require(msg.sender == pendingAdmin, "Not authorized");
    isAdmin[pendingAdmin] = true;
    isAdmin[msg.sender] = false;
    pendingAdmin = address(0);
}
```

---

#### **Shut Off Switch**
**Description:** Pauses all operations for emergencies and allows resumption when resolved.

**Example Code:**
```solidity
bool public isPaused;

function pauseOperations() external onlyAdmin {
    isPaused = true;
}

function resumeOperations() external onlyAdmin {
    isPaused = false;
}
```

---

#### **Set Oracles and Wallet Configurations**
**Description:** Admins configure the addresses for Marketplace Oracles and the Escrow Wallet.

**Example Code:**
```solidity
address public oracleAddress;
address public escrowWallet;

function setOracleAddress(address _oracle) external onlyAdmin {
    oracleAddress = _oracle;
}

function setEscrowWallet(address _wallet) external onlyAdmin {
    escrowWallet = _wallet;
}
```

---

### **Invoice Management**

#### **Invoice Creation**
**Workflow:**
1. Creators input invoice details (amount, expiration, payer wallet).
2. Smart contract validates data and stores it.
3. The system generates a unique invoice ID and payment address.

**Example Code:**
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
    require(_expiration > block.timestamp, "Invalid expiration");

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

#### **Invoice Payment**
**Workflow:**
1. Payers receive a payment URL and log in using WalletConnect.
2. Payments are sent to a unique escrow address.
3. Blockchain Oracles monitor transactions and update the state.

**Example State Updates:**
```solidity
function updateInvoiceState(uint256 _invoiceId, string memory _state) external onlyOracle {
    Invoice storage invoice = invoices[_invoiceId];
    require(invoice.expiration > block.timestamp, "Invoice expired");
    invoice.state = _state;
}
```

---

#### **Refunds**
**Trigger Conditions:**
- Overpayment exceeds the required gas fees.
- Invoice cancellation.
- Marketplace Oracle reallocates funds.

**Example Refund Process:**
```solidity
function refund(uint256 _invoiceId) external {
    Invoice storage invoice = invoices[_invoiceId];
    require(invoice.state == "Cancelled" || invoice.state == "Overpaid", "Invalid state");

    payable(invoice.payer).transfer(invoice.amount);
    invoice.state = "Refunded";
}
```

---

### **Escrow Wallet**

#### **Key Features**
1. **Deterministic Address Generation:**  
   Unique wallet addresses for each invoice, derived securely from an admin-defined key.
   
2. **Admin Configuration:**  
   Flexible wallet configuration for ease of maintenance.

**Planned Enhancements (Phase 2):**
- zk-rollups for improved transaction efficiency and lower gas costs.

---

### **Notifications**

1. **Phase 2:**  
   Notifications sent via Marketplace Oracles for dispute updates and state changes.

2. **Phase 3:**  
   Wallet-based notifications for real-time updates, such as payment confirmations and dispute outcomes.

---

### **Website**

#### **Core Pages**
1. **Landing Page:**  
   Overview, WalletConnect login, and GitHub link.

2. **Pay Invoice Page:**  
   Displays invoice details (requires login to view payment address).

3. **Invoice List:**  
   Searchable and sortable interface for status, creation date, and payer/creator ID.

4. **Admin Dashboard:**  
   Controls for oracles, wallets, and system configuration.
