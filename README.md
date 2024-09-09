<div align="center">
  <a href="https://particle.network/">
    <img src="https://i.imgur.com/xmdzXU4.png" style="display: inline-block; vertical-align: middle;" />
  </a>
  <h3>
    @particle-network/authkit @particle-network/aa Demo Application 
  </h3>
</div>


# Particle Auth, Account Abstraction SDK, Next.js, and ethers V6

⚡️ Basic demo application using `@particle-network/authkit` and `@particle-network/aa` to initiate social login and send transactions via an account abstraction smart account.

This app allows you to log in using social logins and interact with the Ethereum Sepolia and Base Sepolia testnets by displaying account information and sending a transfer transaction to an address you can input in the UI. The user can select to send a gasless transaction or pay gas with the native token.

> This demo guides you through upgrading from the deprecated `auth-core-modal` SDK to the new `AuthKit` SDK. Follow the instructions below to transition this demo app to `AuthKit seamlessly`.

Built using:

- **Particle Auth Core**
- **Particle AA SDK**
- **ethers.js V6.x.x**
- **TypeScript**
- **Tailwind CSS**

## 🔑 Particle Authkit

Particle Authkit enables seamless onboarding to an application-embedded **MPC-TSS/AA** wallet facilitated by social login, such as Google, GitHub, email, phone number, etc.

👉 Learn more about [Particle Auth](https://developers.particle.network/docs/building-with-particle-auth).

## 🪪 Account Abstraction SDK

Particle Network natively supports and facilitates the end-to-end utilization of ERC-4337 account abstraction. This is primarily done through the account abstraction SDK, which can construct, sponsor, and send UserOperations, deploy smart accounts, retrieve fee quotes, and perform other vital functions.

> Every gasless transaction is automatically sponsored on testnet. On mainnet, you'll need to deposit USDT into Paymaster.

👉 Learn more about the [Particle AA SDK](https://developers.particle.network/docs/aa-web-quickstart).

***

👉 Learn more about [Particle Network](https://particle.network).

## 🛠️ Quickstart

### Clone this repository
```
git clone https://github.com/soos3d/particle-auth-aa-demo.git
```

### Move into the app directory (Next JS)

```sh
cd particle-aa-nextjs
```

### Install dependencies

```sh
yarn install
```

Or

```sh
npm install
```

### Set environment variables
This project requires several keys from Particle Network to be defined in `.env`. The following should be defined:
- `NEXT_PUBLIC_PROJECT_ID`, the ID of the corresponding application in your [Particle Network dashboard](https://dashboard.particle.network/#/applications).
- `NEXT_PUBLIC_CLIENT_KEY`, the ID of the corresponding project in your [Particle Network dashboard](https://dashboard.particle.network/#/applications).
-  `NEXT_PUBLIC_APP_ID`, the client key of the corresponding project in your [Particle Network dashboard](https://dashboard.particle.network/#/applications).

### Start the project
```sh
npm run dev
```

Or

```sh
yarn dev
```

Here is an improved version of the README section for upgrading from `auth-core-modal` to `authkit`:

---

## Upgrading to AuthKit in a Next.js Project

To migrate from `auth-core-modal` to `AuthKit`, follow these steps:

### Step 1: Install the Necessary Packages

Run the following command to install the new packages required for the migration:

```sh
yarn add @particle-network/authkit @particle-network/wallet viem@2
```

### Step 2: Create a New `Authkit.tsx` Component

In your `components` directory, create a new file called `Authkit.tsx`:

```tsx
"use client";

// Particle imports
import { AuthType } from "@particle-network/auth-core";
import { sepolia, baseSepolia } from "@particle-network/authkit/chains";
import { AuthCoreContextProvider } from "@particle-network/authkit";
import { EntryPosition } from "@particle-network/wallet";

export const ParticleAuthkit = ({ children }: React.PropsWithChildren) => {
  return (
    <AuthCoreContextProvider
      options={{
        // These environment variables should be defined at runtime
        projectId: process.env.NEXT_PUBLIC_PROJECT_ID!,
        clientKey: process.env.NEXT_PUBLIC_CLIENT_KEY!,
        appId: process.env.NEXT_PUBLIC_APP_ID!,

        chains: [sepolia, baseSepolia], // Configure the chains

        // Limit the available authentication types (remove the array to allow all)
        authTypes: [
          AuthType.email,
          AuthType.google,
          AuthType.twitter,
          AuthType.github,
          AuthType.discord,
          AuthType.phone,
        ],
        themeType: "dark", // Set the theme to dark mode
        fiatCoin: "USD",
        language: "en",

        // Configure Smart Account settings
        erc4337: {
          name: "SIMPLE",
          version: "2.0.0",
        },
        wallet: {
          visible: true, // Set to false to disable the embedded wallet modal
          entryPosition: EntryPosition.TL, // Set the entry position of the wallet modal
          customStyle: {}, // Add custom styling if needed
        },
      }}
    >
      {children}
    </AuthCoreContextProvider>
  );
};
```

### Key Changes in `AuthKit`

- **Chain Imports**: Chains are now imported as `Viem` objects. This is a change from the previous `auth-core-modal` implementation.

### Step 3: Update `layout.tsx`

Import the new `ParticleAuthkit` component into your `layout.tsx` file to integrate it across your app:

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

### Step 4: Update `page.tsx` for Chain and Smart Account Configuration

Modify your `page.tsx` to import the necessary hooks and update the chain objects:

```tsx
import {
  useEthereum,
  useConnect,
  useAuthCore,
} from "@particle-network/authkit";
import { sepolia, baseSepolia } from "@particle-network/authkit/chains";

// Set up and configure the Smart Account
const smartAccount = new SmartAccount(provider, {
  projectId: process.env.NEXT_PUBLIC_PROJECT_ID!,
  clientKey: process.env.NEXT_PUBLIC_CLIENT_KEY!,
  appId: process.env.NEXT_PUBLIC_APP_ID!,
  aaOptions: {
    accountContracts: {
      SIMPLE: [
        {
          version: "2.0.0", // Supports versions 1.0.0 and 2.0.0
          chainIds: [sepolia.id, baseSepolia.id],
        },
      ],
    },
  },
});

// UI Section Example

<h3 className="text-lg mb-2 text-gray-400">
  Chain: {chainInfo.name}
</h3>

<TxNotification
  hash={transactionHash}
  blockExplorerUrl={chainInfo.blockExplorers?.default.url}
/>
```


### Config social logins

List of available social logins:

```sh
{
  email: 'email',
  phone: 'phone',
  facebook: 'facebook',
  google: 'google',
  apple: 'apple',
  twitter: 'twitter',
  discord: 'discord',
  github: 'github',
  twitch: 'twitch',
  microsoft: 'microsoft',
  linkedin: 'linkedin',
  jwt: 'jwt'
}
```

### AA options

You can configure the smart account using the `aaOptions` object in `src/app/page.tsx`.

- **BICONOMY**, a [Biconomy smart account](https://www.biconomy.io/smart-accounts).
  - `version`, either `1.0.0` or `2.0.0`; both versions of Biconomy's smart account implementation are supported.
  - `chainIds`, an array of chain IDs in which the smart account is expected to be used.
- **CYBERCONNECT**, a [CyberConnect smart account](https://wallet.cyber.co/).
  - `version`, currently only `1.0.0` is supported for `CYBERCONNECT`.
  - `chainIds`, an array of chain IDs in which the smart account is expected to be used.
- **SIMPLE**, a [SimpleAccount implementation](https://github.com/eth-infinitism/account-abstraction/blob/develop/contracts/samples/SimpleAccount.sol).
  - `version`, currently only `1.0.0` is supported for `SIMPLE`.
  - `chainIds`, an array of chain IDs in which the smart account is expected to be used.

```ts

import { sepolia, baseSepolia } from "@particle-network/authkit/chains";

  // Set up and configure the smart account
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

  // Use this syntax to upadate the smartAccount if you define more that one smart account provider in accountContracts
  //smartAccount.setSmartAccountContract({ name: "SIMPLE", version: "1.0.0" });
 ```
