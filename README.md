### **Detailed Blockhead Specification**

---

## **Overview**

Blockhead is an open-source, permissionless payment processing platform that operates on the Polygon blockchain. It facilitates trustless cryptocurrency payments, secured by smart contracts, with dispute resolution handled by oracles.

### Core Components:
1. **Website**: A React-based static page hosted on IPFS that manages invoices for users, who can log in using a Polygon-compatible wallet (e.g., MetaMask, WalletConnect).
2. **Smart Contracts**: Solidity-based smart contracts that handle the entire invoicing process, including payment routing, escrow holding, and dispute resolution by Marketplace Oracles.
3. **Escrow Wallet**: A temporary holding wallet for funds before they are released to the recipient.
4. **Marketplace Oracles**: Oracles that facilitate invoice creation, manage invoice hold periods, and resolve disputes.

Example Marketplace: **littlebiggy.net**

---

## **Developer Payments**

### **Total Payment**: $30,000 + Bonus

### **Phase 0: Specification and Platform Setup**

1. Edit and complete the detailed technical specification document.
2. Implement and deploy the landing page wireframe (`blockheadverifybytoken.eth`) on IPFS.
3. Implement wallet login functionality using **WalletConnect** or similar protocols (e.g., MetaMask).
   
**Payment**: $1,500 + Bonus

---

### **Phase 1: Basic Functions and Smart Contracts**

1. Implement basic admin functions.
2. Implement basic functions for **Invoice Creators** (invoice creation, acceptance, and cancellation).
3. Implement basic functions for **Invoice Payers** (invoice payment and release).
   
**Payment**: $5,000  
**Documentation Completion**: $1,000  
**Pass Audit 1**: $3,000  
**Total for Phase 1**: $9,000 + Bonus

---

### **Phase 2: User-Enabled Functions and Smart Contracts**

1. Implement invoice list management (sortable and searchable).
2. Implement overpay and cancel refund functions.
3. Implement marketplace oracle orders and dispute allocations.
4. Implement marketplace pay page API integration.
5. Implement admin shut-off switch, gas fee setting, and fiat conversion.

**Payment**: $5,000  
**Documentation Completion**: $1,000  
**Pass Audit 2**: $3,000  
**Total for Phase 2**: $9,000 + Bonus

---

### **Phase 3: Additional Functions and Contracts**

1. Implement admin transfer functions and a second admin confirmation requirement.
2. Implement ZK rollups.
3. Implement marketplace oracle hold periods for individual invoice creators.
4. Implement user-set invoice terms and additional website features.
5. Implement handling of failed transactions.

**Payment**: Estimated $8,000 + Bonus

---

## **System and Functional Specifications**

### **1. Admin Functions**

Admin functions are accessible through the **Admin Panel** (`blockhead.box/admin`) and control system-wide operations, such as configuring oracles, managing wallets, setting fees, and pausing operations.

#### **Admin Login**
- **Description**: The first admin key and confirmation key are encoded within the smart contract and are used to authenticate the admin user.
- **Example**: The `adminLogin()` function checks that the caller is authorized.

```solidity
function adminLogin(address _adminAddress) public view returns (bool) {
    require(msg.sender == _adminAddress, "Unauthorized: Only the admin can access");
    return true;
}
```

#### **Admin Transfer**
- **Description**: The admin can transfer control to a new admin by confirming the change with a second key.
- **Process**:
  - The admin issues the `transferAdmin` function to set a new admin address.
  - The change must be confirmed by a second key.
- **Example**: 
```solidity
address public adminAddress;

function transferAdmin(address newAdmin) public onlyAdmin {
    require(msg.sender == adminAddress, "Unauthorized: Not the admin");
    adminAddress = newAdmin;
}
```

#### **Shut-off Switch**
- **Description**: The shut-off switch pauses all operational smart contracts, halting invoice creation and payments.
- **Example**:
```solidity
bool public paused = false;

function togglePause() public onlyAdmin {
    paused = !paused;
    emit PauseToggled(paused);
}
```

#### **Configure Oracle**
- **Description**: Set the address of the marketplace oracle that will be responsible for managing invoice creation, dispute resolution, and payment reallocation.
- **Example**:
```solidity
address public marketplaceOracle;

function configureMarketplaceOracle(address oracleAddress) public onlyAdmin {
    marketplaceOracle = oracleAddress;
}
```

#### **Set Gas Fee**
- **Description**: The gas fee is adjustable by the admin to handle extreme network conditions.
- **Example**:
```solidity
uint256 public gasFee;

function setGasFee(uint256 newGasFee) public onlyAdmin {
    gasFee = newGasFee;
}
```

---

### **2. Invoice Creation**

Invoices are created by **Invoice Creators** (typically sellers) on **blockhead.box/invoice/create**.

#### **Invoice Data**:
- **Amount** in MATIC
- **Expiration** (max 180 days)
- **Release Address** (where funds will be sent upon release)
- **Public Key** (to verify identity)

#### **Contract Generation**:
Upon submitting the invoice data, the system generates:
- A unique **Invoice ID**
- A unique **Escrow Wallet Address**
- A **Payment URL** for the **Invoice Payer**

#### **Smart Contract for Invoice Creation**:
```solidity
struct Invoice {
    uint256 amount;
    uint256 expiration;
    address creator;
    address payer;
    address releaseAddress;
    uint256 createdAt;
    string status; // "Created", "Paid", "Accepted", etc.
}

mapping(uint256 => Invoice) public invoices;

function createInvoice(uint256 _amount, uint256 _expiration, address _releaseAddress) public returns (uint256) {
    uint256 invoiceId = uint256(keccak256(abi.encodePacked(block.timestamp, msg.sender, _releaseAddress, _amount)));
    invoices[invoiceId] = Invoice({
        amount: _amount,
        expiration: _expiration,
        creator: msg.sender,
        payer: address(0),
        releaseAddress: _releaseAddress,
        createdAt: block.timestamp,
        status: "Created"
    });
    return invoiceId;
}
```

---

### **3. Invoice Payment**

- **Invoice Payer** receives the invoice URL and logs in with a wallet (e.g., MetaMask).
- **Blockchain Oracle** begins monitoring the blockchain for transactions to the invoice address.

#### **Example: Payment Monitoring**:
```solidity
function payInvoice(uint256 invoiceId) public payable {
    Invoice storage invoice = invoices[invoiceId];
    require(msg.value == invoice.amount, "Incorrect payment amount");
    invoice.status = "Payment Received";
    emit InvoicePaid(invoiceId, msg.sender, msg.value);
}
```

---

### **4. Invoice Acceptance & Cancellation**

- **Invoice Creator** can accept or cancel the invoice.
- **Market Oracle** can also cancel an invoice or reallocate funds if a dispute arises.

#### **Invoice Acceptance**:
The creator accepts the invoice and funds are transferred from the escrow wallet to the release address.

```solidity
function acceptInvoice(uint256 invoiceId) public {
    Invoice storage invoice = invoices[invoiceId];
    require(msg.sender == invoice.creator, "Only the creator can accept the invoice");
    require(keccak256(bytes(invoice.status)) == keccak256(bytes("Payment Received")), "Payment not received");

    // Transfer funds from Escrow wallet to the Invoice Creator's address
    payable(invoice.releaseAddress).transfer(invoice.amount);
    invoice.status = "Accepted";
}
```

---

### **5. Invoice Release**

When conditions are met (payment accepted, hold period exceeded), funds are released to the **Invoice Creator**. If the invoice is disputed, the **Marketplace Oracle** may adjust the release terms.

#### **Example: Invoice Release**:
```solidity
function releaseInvoice(uint256 invoiceId) public {
    Invoice storage invoice = invoices[invoiceId];
    require(keccak256(bytes(invoice.status)) == keccak256(bytes("Accepted")), "Invoice not accepted yet");
    require(block.timestamp >= invoice.createdAt + invoice.expiration, "Hold period not exceeded");

    // Transfer funds from the Escrow Wallet to the Creator
    payable(invoice.releaseAddress).transfer(invoice.amount);
    invoice.status = "Released";
}
```

---

### **6. Refund Functions**

- **Underpayment**: If the payer sends less than the required amount, the invoice can be canceled, or the difference refunded.
- **Overpayment**: Any excess over the required amount (minus transaction fees) is refunded.

#### **Example: Refund Function**:
```solidity
function refundInvoice(uint256 invoiceId) public {
    Invoice storage invoice = invoices[invoiceId];
    require(keccak256(bytes(invoice.status)) == keccak256(bytes("Underpaid")) || keccak256(bytes(invoice.status)) == keccak256(bytes("Cancelled")), "Refund not applicable");

    payable(invoice.payer).transfer(invoice.amount);  // Refund payer
    invoice.status = "Refunded";
}
```

---

### **7. Marketplace Oracles**

**Marketplace Oracles** are responsible for handling more complex interactions:
- Invoice creation
- Dispute resolution
- Hold periods

#### **Marketplace Oracle Invoice Creation**:
```sol

idity
function createMarketplaceInvoice(address _payer, uint256 _amount, uint256 _expiration) public {
    // Oracle logic for creating and assigning invoice
}
```

---

## **Website Structure**

The website provides the front-end interface for interaction with the Blockhead system.

### **Pages**:
1. **Landing Page**: Displays general information about Blockhead and a login option.
2. **Invoice Creation**: Form for creating a new invoice.
3. **Invoice List**: Displays all the user's invoices with statuses.
4. **Invoice Pay Page**: Allows the payer to settle the invoice by sending payment.

### **UI Components**:
- **Navbar**: Includes links to create invoices, view invoices, admin (conditional on wallet key).
- **Invoice Management**: For creators to manage their invoices, including accepting, canceling, or viewing status.

---

## **Escrow Wallet**

The **Escrow Wallet** holds payments until release conditions are met. Each invoice is assigned a unique wallet address generated from the admin's settings.

---

## **Notifications**

Notifications are sent via:
- **Marketplace Oracles**: For dispute resolutions and invoice reallocation.
- **Wallet Messaging**: For other user actions (e.g., invoice updates).
