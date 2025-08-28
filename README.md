<div align="center">
  <a href="https://particle.network/">
    <img src="https://i.imgur.com/xmdzXU4.png" style="display: inline-block; vertical-align: middle;" />
  </a>
  <h3>
    @particle-network/authkit @particle-network/aa Demo Application 
  </h3>
</div>

# Migrating from `auth-core-modal` to AuthKit

This repository demonstrates how to **upgrade from the deprecated `auth-core-modal` package to the new `@particle-network/authkit` SDK**.  
It includes a working Next.js demo showing you:

- How to replace your existing `auth-core-modal` setup with `AuthKit`.  
- How to configure social logins, chain support, and smart accounts.  
- How to send transactions using the `@particle-network/aa` SDK with ethers v6.  

By following this guide, you’ll learn the exact changes needed to modernize your project and fully adopt AuthKit.

---

## 🔑 Particle AuthKit

Particle AuthKit enables application-embedded onboarding into an **MPC-TSS/AA** wallet via social login (Google, GitHub, email, phone number, etc.).

👉 Learn more about [Particle Auth](https://developers.particle.network/social-logins/auth/introduction).

## 🪪 Account Abstraction SDK

Particle Network provides native support for ERC-4337 account abstraction. The AA SDK allows you to construct, sponsor, and send UserOperations, deploy smart accounts, retrieve fee quotes, and more.

> On testnets, every gasless transaction is automatically sponsored. On mainnet, you’ll need to deposit USDT into Paymaster.

👉 Learn more about the [Particle AA SDK](https://developers.particle.network/aa/introduction).

---

# 🛠️ Upgrading to AuthKit

The steps below walk you through migrating this demo (and your own apps) from `auth-core-modal` to `@particle-network/authkit`.

### Step 1: Install the New Packages

```sh
yarn add @particle-network/authkit @particle-network/wallet viem@2
```

### Step 2: Create a New `Authkit.tsx` Component

In your components directory:

```tsx
"use client";

import { AuthType } from "@particle-network/auth-core";
import { sepolia, baseSepolia } from "@particle-network/authkit/chains";
import { AuthCoreContextProvider } from "@particle-network/authkit";
import { EntryPosition } from "@particle-network/wallet";

export const ParticleAuthkit = ({ children }: React.PropsWithChildren) => {
  return (
    <AuthCoreContextProvider
      options={{
        projectId: process.env.NEXT_PUBLIC_PROJECT_ID!,
        clientKey: process.env.NEXT_PUBLIC_CLIENT_KEY!,
        appId: process.env.NEXT_PUBLIC_APP_ID!,

        chains: [sepolia, baseSepolia],

        authTypes: [
          AuthType.email,
          AuthType.google,
          AuthType.twitter,
          AuthType.github,
          AuthType.discord,
          AuthType.phone,
        ],
        themeType: "dark",
        fiatCoin: "USD",
        language: "en",

        erc4337: {
          name: "SIMPLE",
          version: "2.0.0",
        },
        wallet: {
          visible: true,
          entryPosition: EntryPosition.TL,
          customStyle: {},
        },
      }}
    >
      {children}
    </AuthCoreContextProvider>
  );
};
```

> 🔑 Key change: Chains are now imported as Viem objects instead of the old format.

⸻

### Step 3: Wrap Your App in `layout.tsx`

```tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

import { ParticleAuthkit } from "./components/AuthKit";

export const metadata: Metadata = {
  title: "Particle Auth App",
  description: "An application leveraging Particle Auth for social logins.",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <ParticleAuthkit>{children}</ParticleAuthkit>
      </body>
    </html>
  );
}
```

⸻

### Step 4: Update page.tsx

Update hooks and smart account configuration:

```
import {
  useEthereum,
  useConnect,
  useAuthCore,
} from "@particle-network/authkit";
import { sepolia, baseSepolia } from "@particle-network/authkit/chains";

const smartAccount = new SmartAccount(provider, {
  projectId: process.env.NEXT_PUBLIC_PROJECT_ID!,
  clientKey: process.env.NEXT_PUBLIC_CLIENT_KEY!,
  appId: process.env.NEXT_PUBLIC_APP_ID!,
  aaOptions: {
    accountContracts: {
      SIMPLE: [
        {
          version: "2.0.0",
          chainIds: [sepolia.id, baseSepolia.id],
        },
      ],
    },
  },
});

// UI example
<h3 className="text-lg mb-2 text-gray-400">Chain: {chainInfo.name}</h3>
<TxNotification
  hash={transactionHash}
  blockExplorerUrl={chainInfo.blockExplorers?.default.url}
/>
```
⸻

### Configuring Social Logins

```tsx
Available options:

{
  "email": "email",
  "phone": "phone",
  "facebook": "facebook",
  "google": "google",
  "apple": "apple",
  "twitter": "twitter",
  "discord": "discord",
  "github": "github",
  "twitch": "twitch",
  "microsoft": "microsoft",
  "linkedin": "linkedin",
  "jwt": "jwt"
}
```

⸻

Configuring Smart Accounts (aaOptions)

Examples:
	•	BICONOMY — versions 1.0.0 and 2.0.0 supported.
	•	CYBERCONNECT — currently only 1.0.0.
	•	SIMPLE — supports 1.0.0 and 2.0.0.

```tsx
import { sepolia, baseSepolia } from "@particle-network/authkit/chains";

const smartAccount = new SmartAccount(provider, {
  projectId: process.env.REACT_APP_PROJECT_ID!,
  clientKey: process.env.REACT_APP_CLIENT_KEY!,
  appId: process.env.REACT_APP_APP_ID!,
  aaOptions: {
    accountContracts: {
      SIMPLE: [
        {
          version: "2.0.0",
          chainIds: [sepolia.id, baseSepolia.id],
        },
      ],
    },
  },
});

// Switch implementation if multiple contracts are defined
// smartAccount.setSmartAccountContract({ name: "SIMPLE", version: "1.0.0" });
```

⸻

##  Quickstart (Demo App)

1. Clone the repository

```sh
git clone https://github.com/soos3d/particle-auth-aa-demo.git
```

2. Move into the app directory

```sh
cd particle-aa-nextjs
```
3. Install dependencies

```
yarn install
# or
npm install
```

4. Set environment variables

Define the following in .env:
	•	NEXT_PUBLIC_PROJECT_ID — from the Particle dashboard.
	•	NEXT_PUBLIC_CLIENT_KEY — from the dashboard.
	•	NEXT_PUBLIC_APP_ID — from the dashboard.

5. Start the project

```sh
npm run dev
# or
yarn dev
```
