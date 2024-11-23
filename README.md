
***
# Overview

Blockhead is building an open source, permissionless and reputation bound payment processor on polygon. payments will initially be made in matic.

Blockhead enables safe crypto payments with no-trust code and dispute settlement by oracle. Blockhead is permissionless yet effectively bound by by the reputation data held in the oracles.

At the heart of the system is:

`the website` blockhead.box, an ipfs hosted react static page website that manages invoices for users logged in with a polygon compatible wallet.

`the smart contracts` solidity contracts that direct payments to parties in accordance with the Marketplace Oracles

`the Escrow Wallet` where payments are stored before release

`Marketplace oracles` which:

i) deliver invoice data to the smart contracts

ii) determine escrow hold periods for Invoice Creators

iii) determine post-dispute allocations.

example market: littlebiggy.net

`Blockchain Oracles` which monitor blockchains for updates on payments

***


# Developer Payments

### **$30,000 + bonus**

## **Phase 0: specification and platform** 

edit and complete this specification

establish blockheadverifybytoken.eth ipfs site wireframe landing page

establish wallet login

1.5k+bonus



## **Phase 1: basic functions and accompanying smart contracts added to wireframe**

basic admin functions

logged in creator : invoice creation

  invoice acceptance

  invoice cancellation

logged in payer: invoice payment

invoice release

$5,000

documentation completion: $1,000

pass audit 1 : $3,000

total - $9,000 + bonus



## **Phase 2 user enabling functions and smart contracts added** 

invoice list

overpay refund fxs

cancel refund fxs

marketplace oracle orders

marketplace oracle dispute allocations

marketplace pay page api integration

admin shut off switch

set gas fee

fiat conversion

$5,000

documentation complete: $1,000

pass audit 2 : $3,000

$9,000 + bonus total



## **Phase 3 additional functions, contracts**

transfer admin

2nd admin confirmation required

zk rollups

marketplace oracle sets hold periods for individual invoice creators

user set terms

more site fxs

disposals from failed transactions

pass audit 3

estimated $8,000 +bonus


***

# Functions

Business rules and their related functions are contextualized through the basic actions of Admin, Seller, and Buyer users.

## Admin Functions

admin functions are controlled from blockhead.box/admin


`Admin Login` the first admin key and confirmation key is encoded within the smart contract.

`Transfer admin` the admin key and confirmation key can be changed with a 2nd confirmation

`Shut off Switch` pauses all operations

`Configure Oracle` enters address for marketplace oracle

`2nd admin confirmation` additional signature for certain admin fxs

`Configure Escrow Address` sets address for escrow wallet

`Configure Blockhead Fee Margin %`

`Set Gas Fee` (a control is always necessary under extreme network conditions)

`Set Blockhead Wallet Address` for paying gas fees and collecting blockhead fees

`Hold Time` sets time to release for all contracts.

individual holds for invoice creators from market oracle **(phase 2)**

`Set Market Oracle`

`Blacklist` sets wallet addresses that can not create invoices

`Trigger Events` resend interrupted transactions

`Set Circuit Breakers` in smart contract

***


## Invoice Creation

The Invoice Creator is typically a seller of goods/services; the Invoice Payer is a corresponding buyer.

Sequence: to build an invoice @blockhead.box/invoice/create

Creator enters Invoice data:

* Amount in matic

* Expiration - entry limit 180 days

Additional data is captured from Creator's wallet:

* Release Address

* Public Key

Additional data derived from admin configuration is added to contract:

* Release Date

* Escrow Wallet Address

* Market Oracle (If invoice created by Market Oracle)

Upon validation The Smart Contract generates and displays on page:

* A unique Invoice ID

* A unique Payment Address from the Escrow Wallet (these can be identifiers of our own or simply the associated keys/wallet addresses)

* A blockhead.box/invoice/pay url and QR

The Blockchain Oracle then begins querying the blockchain for payments until expiry

Creators then shares the invoice url with Payer

***


## Marketplace Oracles

`Preferred Oracle Network:  Chainlink (permissionless)`

1) Marketplace Oracles create invoices on Blockhead including all the fields that an individual Invoice Creator would submit + conditions text + invoice hold period (otherwise a blockhead admin configuration) + Invoice Payer Address

2) Invoices created by a Marketplace Oracle can be disputed before release. In such cases the Marketplace Oracle can change release timing and re-allocate funds from the Release to the Invoice Payer Address.

***


## Invoice Payment

Sequentially: Invoice Payer has

* received a url for invoice payment from the Invoice Creator

* logged in with their wallet (not required for Marketplace Invoices which arrive via the Marketplace Oracle )

Smart Contract and its related entities have

* generated a unique invoice address from the Escrow Wallet

* begun monitoring for payments via the Blockchain Oracle

* (If Marketplace Invoice) emitted an event sending the payment url to the Marketplace

The Invoice Payment
The Blockchain Oracle monitors the blockchain and along with commands from the website sets appropriate state changes from Created to:

* Payment Received -> awaits Invoice Creator command: Accept Payment.

* Depending upon the amount received the state can change to:

* Underpaid

* Overpaid

Creator can accept invoices paid within 3 days, then the Invoice state changes to Refund Pending.
Payments received after cancellation are refunded.

***


## Invoice Acceptance & Cancellation

Creators, Oracles and system rules evoke state changes for Invoices:

Paid

Accepted

Cancelled by Creator

Cancelled by Marketplace Oracle (buyer or seller can cancel an order)

Awaiting Marketplace Oracle Allocation (when a dispute is being settled within the marketplace)

Cancelled by Timeout Creator

Cancelled by Timeout Payer

***


## Invoice Release

Invoice releases are the executions of our smart contracts.



Conditions required for release from Escrow Wallet to Invoice Creator:

Invoice accepted by Invoice Created

Hold period exceeded



Conditions required for release from escrow wallet to Invoice payer:

Refund state from:

paid invoice not accepted

invoice overpayment amount

late payment



Conditions required for Execution by Marketplace Oracle

Invoice Disputed

***


## Under and Over Payments

Underpayments can be Accepted by the Creator or the invoice can be cancelled

Overpayments greater than required gas fees are automatically refunded to the Payer's address

***


## Disputed Invoices

**Phase 2**

Unreleased Invoices (anything still in escrow/awaiting release) are subject to release re-allocation between Creator and Payer by the Marketplace Oracle, ostensibly in the case of a dispute.

***


## Refund Functions

Refund functions are applied immediately when 

* an invoice is underpaid by an amount in excess of gas fees 

* cancelled by either party.

* funds are allocated to Payer from oracular re-allocation

Refunds are sent to the Payer's sending address. 

***


## Invoice List

Logged in Creators or Payers access:

a list with sortable headings:

status, (date) created, (date) paid, creator id payer id

and a search function by:

invoice id, invoice address, state, creator id, payer id

***


## Website

blockhead.box is a react static page website, hosted on ipfs, 

`wallet login provider:` wallet connect

`hosting:` fleek or 4everland

`test server:` vercel

`style resources:` bootstrap, right sidebar responsive site.  basic styling until phase 3

`.box domain management:` my.box and 3dns.box



`the default landing page:` requires explanation text, login, link to our github

`pay invoice page:` Invoice Payers are given a url from Invoice Creators to land on this page which shows all of the invoice details except the pay to address which requires login/wallet connection

`navbar:` links to create invoice, invoice list, find, admin (conditional to wallet key)

`invoice list`

`create invoice page`

`invoice page:` displays current state and history, creators can cancel invoice here

`invoice pay page:` note login is still required to display this direct linked page

`admin page`

***


## Escrow Wallet

The Escrow wallet holds payments until release; the escrow wallet is set by the admin. each invoice payment receives a new address for this wallet.

The receiving address for the wallet is deterministic. The addresses are derived from a key set by the admin and subject to security limitations in **(phase 2)**

***


## Notifications

Through Marketplace Oracle (phase 2)

Through Wallet Messaging (phase 3)


***
