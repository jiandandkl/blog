---
uuid: c4246980-61de-11f0-bd10-237001c20106
author: dujun
email: emolingzhu@126.com
github: https://github.com/dujun
avatar: https://avatars.githubusercontent.com/u/119213?v=4
title: A web3 introductory project
date: 2025-07-15 18:49:43
tags:
---

## Overview

Recently, a friend is considering finding a remote Web3 job. I've also seen people asking about Web3 entry-level questions on V2EX. Since my previous Web3 project ended, I've organized Web3-related code to share, hoping to help everyone get a general understanding of contract interactions.

### Main Features

- Wallet connections for OKX, MetaMask, Phantom, and other wallets
- Multi-chain contract interactions for EVM, Tron, Solana, and other chains

### Project Structure

```
web3-start/
├── src/
│   ├── components/           # Frontend interaction components (wallet interaction, trading panels, etc.)
│       ├── Interaction/      # Contract interaction feature pages
│       ├── SelectChain/      # Chain selection page
│       ├── SelectWallet/     # Wallet selection page
│   ├── constants/            # Chain configurations and constant definitions
│   ├── contracts/            # Smart contract source code
│   ├── hook/                 # Business logic hooks (wallet connection, transaction processing, balance queries, etc.)
│   ├── lib/                  # Blockchain interaction libraries (EVM transactions, etc.)
│   ├── store/                # State management (wallet, balance, user info, etc.)
│   ├── types/                # TypeScript type declarations
├── public/                   # Public resources directory
├── .env                      # Environment variables (API keys and other configurations)
├── README.md                 # Project documentation
├── package.json              # Dependencies and script configurations
```

## Detailed Wallet Interaction Implementation

The project supports multiple wallet connections (such as MetaMask, OKX, TronLink, Sui Wallet) and account management. The core logic is mainly distributed in `src/hook/useConnectWallet.ts` and related components. The following will use EVM wallets as an example to analyze the implementation of wallet connection and on-chain transactions in detail.

### 1. Wallet Connection (Using EVM/MetaMask as Example)

Wallet connection is mainly achieved by calling the wallet injected into the browser (such as window.ethereum) and requesting account authorization:

```typescript name=src/hook/useConnectWallet.ts
const connectEvmWallet = async () => {
  try {
    const provider = getProvider();
    // Request user authorization and connect wallet
    const { accounts } = await connectAsync({
      connector: provider,
      chainId: CHAIN[chainType.toUpperCase()].id,
    });
    await handleConnectSuccess(accounts[0]);
  } catch (error) {
    handleConnectError(error);
  }
};
```

For OKX, Tron, Sui, and other wallets, there are also dedicated connection methods, for example:

```typescript
const connectTronWallet = async () => {
  const currentProvider = window?.tronLink;
  subscribeTronWallet();
  window?.tron.request({ method: "eth_requestAccounts" }).then((res: any) => {
    addressRef.current = currentProvider.tronWeb.defaultAddress.base58;
    handleSignMessage(
      currentProvider.tronWeb.defaultAddress.base58,
      CHAIN.TRON.brief
    );
  });
};
```

### 2. Wallet Event Listening

After connecting the wallet, you need to listen for events such as account switching and disconnection:

```typescript name=src/hook/useConnectWallet.ts
const subscribeSuiWallet = async () => {
  const provider = window.okxwallet.sui;
  provider.features["standard:events"].on("connect", () =>
    setIsConnected(true)
  );
  provider.features["standard:events"].on(
    "accountChanged",
    (publicKey: any) => {
      if (publicKey) {
        console.log(`Switched to account ${publicKey.toBase58()}`);
      }
    }
  );
  provider.features["standard:events"].on("disconnect", () => {
    // disconnect();
  });
};
```

## Balance Queries and On-Chain Interactions

After successful wallet connection, you can get the current account balance and display it on the frontend page:

```typescript name=src/store/balance.ts
const fetchBalanceLogic = async (set: any, chain: any, address: string) => {
  set({ isLoading: true });
  let balance: any = null;

  if (isEvmChain(chain.brief)) {
    balance = await getEvmBalance(chain.brief, address);
    set({
      balance: {
        origin: balance,
        value: formatUnits(balance, chain.decimals),
      },
    });
    return;
  }

  // Other chains (such as Solana, Tron) handling methods
  // ...
};
```

## Contract Interactions and Transaction Flow

The frontend initiates transactions through wallet signatures to complete on-chain operations such as transfers. The typical flow is as follows:

### 1. Initiating Transactions

In `src/components/Interaction/index.tsx`, users can initiate transactions by entering an amount and clicking the button:

```tsx name=src/components/Interaction/index.tsx
const handleTrade = async () => {
  if (!inputValue || Number(inputValue) <= 0) {
    addToast({ title: "Please enter a valid amount", color: "danger" });
    return;
  }
  // Check if balance is sufficient
  if (Number(inputValue) >= 1) {
    addToast({ title: "Please enter a amount < 1", color: "danger" });
    return;
  }

  try {
    const voteTokenAmount = parseUnits(
      inputValue,
      CHAIN[chain.brief!.toUpperCase()]?.decimals ?? 18
    );
    // ...Additional validation and authorization logic
    setLoading(true);

    await handleTradeFunc({
      inputValue: Number(inputValue),
      address,
      voteToken: {
        tokenAddress: "",
        chainType: chain.brief,
        tokenDecimals: CHAIN[chain.brief!.toUpperCase()]?.decimals,
      },
      mainTokenBalance: balance.origin,
    });

    setShowTradeSuccess(true);
    setTimeout(() => setShowTradeSuccess(false), 2000);
  } catch (error: any) {
    // ...Error handling
  } finally {
    setLoading(false);
  }
};
```

### 2. EVM Chain Contract Interactions (Signing & Sending Transactions)

In `src/lib/okxEvm.ts`, the `viem` library and wallet signing are used to implement transaction sending:

```typescript name=src/lib/okxEvm.ts
const publicClient = createPublicClient({ chain, transport: http() });
const { request } = await publicClient.simulateContract({
  address: params.smartContractAddress as `0x${string}`,
  abi: EvmVoteAbi,
  functionName: "placeVote",
  args: [
    [
      params.voteId,
      params.tokenAddress,
      params.tokenAmount,
      params.expireTimestamp,
      params.signature,
    ],
  ],
  value: parseUnits(params.tokenAmount, 0),
  account: params.senderAddress as `0x${string}`,
});

// Send transaction
const walletClient = createWalletClient({
  chain,
  transport: custom(window?.ethereum),
});
const result = await walletClient.writeContract(request);
```

## Source Code and Live Demo

GitHub Repository: https://github.com/jiandandkl/web3-start

Live Demo: https://web3-start-sandy.vercel.app/
