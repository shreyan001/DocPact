# DocPact: A Decentralized, Agent-Driven Trust Layer for Secure Digital Exchange

> **Note:** For detailed project information, technical walkthrough, and use cases, see [description.md](./description.md)
> 
> **Technical Architecture:** For comprehensive technical architecture, implementation details, and system design, see [ARCHITECTURE.md](./ARCHITECTURE.md)

DocPact is a Filecoin-powered, AI-mediated, and privacy-preserving "digital third party" for secure, verifiable, and flexible digital asset exchange. This repository contains the Next.js frontend application that enables creators, teams, businesses, and organizations to exchange digital content without requiring trust in counterparties or platform operators.

## 🚀 Quick Start

### Prerequisites

- **Node.js** (v18 or higher)
- **npm** or **yarn** package manager
- **MetaMask** or compatible Web3 wallet
- Access to **Filecoin Calibration** testnet (for development)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/shreyan001/DocPact.git
   cd DocPact
   ```

2. **Install dependencies:**
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Configure environment (optional):**
   - The app uses default configuration from `config.ts`
   - Storage capacity: 10GB
   - Persistence period: 30 days
   - CDN enabled for faster retrieval

### Development

1. **Start the development server:**
   ```bash
   npm run dev
   # or
   yarn dev
   ```

2. **Open your browser:**
   Navigate to [http://localhost:3000](http://localhost:3000)

3. **Connect your wallet:**
   - Ensure MetaMask is installed
   - Switch to Filecoin Calibration testnet
   - Connect your wallet through the app interface

### Production Build

1. **Build the application:**
   ```bash
   npm run build
   # or
   yarn build
   ```

2. **Start production server:**
   ```bash
   npm start
   # or
   yarn start
   ```

## 🔧 Configuration

### Storage Configuration

Edit `config.ts` to customize storage settings:

```typescript
export const config = {
  storageCapacity: 10,      // GB of storage capacity
  persistencePeriod: 30,    // Days of lockup period
  minDaysThreshold: 10,     // Minimum days threshold
  withCDN: true,           // Enable CDN for faster retrieval
};
```

### Network Configuration

The app is configured to work with:
- **Filecoin Mainnet** (production)
- **Filecoin Calibration** (testnet/development)

Network configuration is handled automatically through Wagmi and RainbowKit.

## 🛠 Tech Stack

- **Frontend:** Next.js 15, React 19, TypeScript
- **Styling:** Tailwind CSS 4
- **Web3:** Wagmi, RainbowKit, Ethers.js, Viem
- **Storage:** Filoz Synapse SDK
- **Animations:** Framer Motion
- **State Management:** TanStack React Query

## 📦 Key Dependencies

- `@filoz/synapse-sdk` - Filecoin storage integration
- `@rainbow-me/rainbowkit` - Wallet connection UI
- `wagmi` - React hooks for Ethereum
- `ethers` - Ethereum library
- `next` - React framework
- `framer-motion` - Animation library

## 🌐 Deployment

### Vercel (Recommended)

1. **Connect your repository to Vercel**
2. **Configure build settings:**
   - Build Command: `npm run build`
   - Output Directory: `.next`
3. **Deploy**

### Other Platforms

The app can be deployed on any platform that supports Next.js:
- Netlify
- Railway
- Heroku
- AWS Amplify
- Self-hosted with PM2

## 🔐 Security & Privacy

- **Client-side encryption** for all file uploads
- **Wallet-based authentication** (no traditional accounts)
- **Decentralized storage** on Filecoin network
- **Privacy-preserving** agent verification

## 🧪 Development Scripts

```bash
npm run dev      # Start development server with Turbopack
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## 📄 License

This project is open source. See the [LICENSE](LICENSE) file for details.

## 🔗 Links

- **Detailed Documentation:** [description.md](./description.md)
- **Filoz Synapse SDK:** [NPM Package](https://www.npmjs.com/package/@filoz/synapse-sdk)
- **Filecoin Network:** [Official Website](https://filecoin.io/)
- **RainbowKit:** [Documentation](https://www.rainbowkit.com/)

## 🆘 Support

If you encounter any issues:
1. Check the [description.md](./description.md) for detailed technical information
2. Ensure your wallet is connected to the correct network
3. Verify you have sufficient testnet tokens for transactions
4. Open an issue in this repository

---

**DocPact is the programmable, reliable, decentralized agent for digital value exchange. Powered by Filecoin, extensible by design, and robust enough to power the digital work, knowledge, and creativity of tomorrow.**