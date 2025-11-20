#  Beginner Walkthrough: Using Arc’s USDC (Native + ERC-20) in a Dapp
Arc gives you **one USDC**, but with **two ways to interact with it**:

•	**Native balance** (18-decimals) → used for gas + low-level transfers
  
  •	**ERC-20 interface** (6-decimals) → used for normal token logic (transfer, approve, balanceOf)

You’ll use **both**, depending on what your dapp needs.
 
 ## Overview

Arc’s USDC design is unique: it’s one token with two interfaces. Developers can interact with it either as a native balance (18 decimals) for paying transaction fees and low-level operations, or as an ERC-20 token (6 decimals) for normal dapp logic like transfers, approvals, and displaying balances.

This dual-interface system simplifies the user experience users only hold one tokenw hile preserving compatibility with standard Ethereum tooling. The key for developers is knowing when to use native vs ERC-20 and handling the decimal differences correctly.

# Table of Contents

1. [Install Basics](#install-basics)  
2. [Set Up Provider and Wallet](#set-up-provider-and-wallet)  
3. [Read Native USDC Balance](#read-native-usdc-balance)  
4. [Read ERC-20 USDC Balance](#read-erc-20-usdc-balance)  
5. [Convert Between Native and ERC-20](#convert-between-native-and-erc-20)  
6. [ERC-20 USDC Transfer](#erc-20-usdc-transfer)  
7. [Native USDC Transfer](#native-usdc-transfer)  
8. [When to Use Which](#when-to-use-which)  
9. [Common Beginner Mistakes](#common-beginner-mistakes)  
10. [Made By](#made-by)

---

## Install Basics
```bash
npm install ethers
````
## Set Up Provider and Wallet
```bash
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider("https://rpc.testnet.arc.network"); // example RPC
const wallet = new ethers.Wallet(PRIVATE_KEY, provider);
````
## Read Native USDC Balance
This is the “gas balance” and raw chain balance.
```bash
const rawNative = await provider.getBalance(wallet.address);
console.log("Native USDC (18-dec):", rawNative.toString());
````
Use this for anything related to **fees**, **system calls**, or **low-level transfers**
## Read ERC-20 USDC Balance
Arc shows an ERC-20 interface for app logic
```bash
const usdcAddress = "0x4200…"; // Arc’s USDC contract address
const usdcAbi = [
  "function balanceOf(address) view returns (uint256)",
  "function transfer(address,uint256) returns (bool)",
];

const usdc = new ethers.Contract(usdcAddress, usdcAbi, wallet);

const rawErc20 = await usdc.balanceOf(wallet.address);
console.log("ERC-20 USDC (6-dec):", rawErc20.toString());
````
Use this for normal **token transfers**, **UI balances**, **approvals**, and **dapp logic**
## Convert Between Native and ERC-20
Arc syncs these under the hood, but **you must handle decimals correctly**.
```bash
// Convert native → ERC20 representation
const asERC20 = rawNative / 1e12;

// Convert ERC20 → native representation
const asNative = rawErc20 * 1e12;
````
Think of it like:

•	1 native unit = 10⁻¹⁸
  
•	1 ERC20 unit = 10⁻⁶

Arc’s precompile keeps the actual value identical, you’re only switching how it’s represented.
## ERC-20 USDC Transfer
Make a simple USDC transfer (ERC-20)
```bash
const tx = await usdc.transfer("0xRecipient...", 5_000_000); 
// 5 USDC because 6 decimals

await tx.wait();
console.log("Sent 5 USDC!");
````
## Native USDC Transfer
Most dapps won’t need this, but it’s good to understand.
```bash
const tx = await wallet.sendTransaction({
  to: "0xRecipient...",
  value: ethers.parseUnits("5", 18), // 5 USDC
});

await tx.wait();
console.log("Native transfer complete!");
````
Native transfers behave like ETH transfers on Ethereum but the asset is USDC
## When to Use Which
 | Action                     | Use Native (18-dec) | Use ERC-20 (6-dec) |
|----------------------------|-------------------|------------------|
| Pay gas                    | ✅                | ❌               |
| Display balances in UI     | ❌                | ✅               |
| Dapp token transfers       | ❌                | ✅               |
| Low-level system functions | ✅                | ❌               |
| Approvals / allowances     | ❌                | ✅               |
## Common Beginner Mistakes

•	Mixing 18-decimal and 6-decimal amounts
  
•	Doing frontend math with JS floats (always use BigInt/string)
  
•	Calling native balance for UI (users expect ERC-20 formatting)
  
•	Assuming they are two separate tokens they’re one, with two views

## made by
{qwerty12345671}
  
