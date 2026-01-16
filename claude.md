# Polymarket Copy Trading Bot

A bot that monitors Polymarket traders and automatically copies their trades to your wallet.

## Overview

This bot watches one or more Polymarket traders and mirrors their trading activity:
- When a tracked trader **buys** a position, the bot buys the same outcome
- When a tracked trader **sells** a position, the bot sells the same outcome
- Trade sizes are scaled based on your configured strategy (percentage, fixed, or adaptive)

### Safety Features
- **Preview mode**: Test without executing real trades
- **Order size limits**: Cap maximum trade size
- **Position limits**: Prevent over-concentration in single markets
- **Daily volume limits**: Control total daily trading activity
- **Graceful shutdown**: Properly handles interrupts

## Quick Start

```bash
# 1. Install dependencies
npm install

# 2. Copy environment template
cp .env.example .env

# 3. Edit .env with your configuration (see Configuration section)

# 4. Run health check to verify setup
npm run health-check

# 5. Start in preview mode first (set PREVIEW_MODE=true in .env)
npm run dev

# 6. Once satisfied, set PREVIEW_MODE=false and restart
```

## Configuration Reference

### Required Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `PRIVATE_KEY` | Your wallet private key (without 0x prefix) | `abc123...` |
| `PROXY_WALLET` | Your wallet address (must match private key) | `0x1234...` |
| `USER_ADDRESSES` | Trader address(es) to copy | `0xabcd...` |
| `RPC_URL` | Polygon RPC endpoint | `https://polygon-mainnet.infura.io/v3/YOUR_KEY` |
| `MONGO_URI` | MongoDB connection string | `mongodb+srv://...` |

### Copy Strategy Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `COPY_STRATEGY` | Strategy type: `PERCENTAGE`, `FIXED`, or `ADAPTIVE` | `PERCENTAGE` |
| `COPY_SIZE` | Main parameter (% for PERCENTAGE, $ for FIXED) | `10.0` |
| `MAX_ORDER_SIZE_USD` | Maximum single order size | `100.0` |
| `MIN_ORDER_SIZE_USD` | Minimum order size (Polymarket requires $1+) | `1.0` |

### Copy Strategies Explained

**PERCENTAGE** (Recommended for beginners)
- Copies a fixed percentage of the trader's order
- Example: `COPY_SIZE=10.0` means copy 10% of trader's orders
- Trader buys $500 -> You buy $50

**FIXED**
- Copies a fixed dollar amount regardless of trader's order size
- Example: `COPY_SIZE=25.0` means always copy $25
- Predictable spending per trade

**ADAPTIVE**
- Dynamically adjusts percentage based on trade size
- Uses `ADAPTIVE_MIN_PERCENT`, `ADAPTIVE_MAX_PERCENT`, `ADAPTIVE_THRESHOLD_USD`
- Larger trades get lower %, smaller trades get higher %

### Tiered Multipliers (Advanced)

For traders with varying position sizes, use tiered multipliers:

```env
# Format: "min-max:multiplier,min-max:multiplier,min+:multiplier"
TIERED_MULTIPLIERS=1-100:2.0,100-1000:0.5,1000-10000:0.1,10000+:0.01
```

This applies different multipliers based on the trader's order size:
- $50 order × 2.0 = $100 copy
- $500 order × 0.5 = $250 copy
- $5000 order × 0.1 = $500 copy

### Other Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `FETCH_INTERVAL` | Seconds between activity checks | `1` |
| `TOO_OLD_TIMESTAMP` | Ignore trades older than X hours | `1` |
| `PREVIEW_MODE` | If `true`, monitor only without executing | `false` |
| `TRADE_AGGREGATION_ENABLED` | Combine small trades | `false` |

## Available Scripts

### Core Operations

| Script | Command | Description |
|--------|---------|-------------|
| Setup | `npm run setup` | Interactive configuration wizard |
| Start (dev) | `npm run dev` | Run bot in development mode |
| Start (prod) | `npm run start` | Run compiled bot |
| Health Check | `npm run health-check` | Verify configuration and connectivity |

### Account Management

| Script | Command | Description |
|--------|---------|-------------|
| Check Stats | `npm run check-stats` | View your trading statistics |
| Check Allowance | `npm run check-allowance` | Check USDC approval status |
| Set Allowance | `npm run set-token-allowance` | Approve USDC for trading |
| Manual Sell | `npm run manual-sell` | Manually sell a position |
| Sell Large | `npm run sell-large` | Sell positions above threshold |

### Position Management

| Script | Command | Description |
|--------|---------|-------------|
| Close Stale | `npm run close-stale` | Close old positions |
| Close Resolved | `npm run close-resolved` | Close resolved market positions |
| Redeem Resolved | `npm run redeem-resolved` | Redeem winnings from resolved markets |

### Trader Discovery

| Script | Command | Description |
|--------|---------|-------------|
| Find Traders | `npm run find-traders` | Discover profitable traders |
| Find Low Risk | `npm run find-low-risk` | Find conservative traders |
| Scan Traders | `npm run scan-traders` | Batch scan multiple traders |
| Scan Markets | `npm run scan-markets` | Find traders from active markets |

### Analysis & Debugging

| Script | Command | Description |
|--------|---------|-------------|
| Simulate | `npm run simulate` | Backtest strategy on trader |
| Check Activity | `npm run check-activity` | View recent trader activity |
| Check PnL | `npm run check-pnl` | Analyze profit/loss |
| Audit | `npm run audit` | Audit copy trading algorithm |

## Architecture

```
src/
├── index.ts              # Entry point, starts monitor & executor
├── config/
│   ├── env.ts            # Environment variable loading
│   ├── db.ts             # MongoDB connection
│   └── copyStrategy.ts   # Copy strategy logic
├── services/
│   ├── tradeMonitor.ts   # Watches trader activity
│   ├── tradeExecutor.ts  # Executes copy trades
│   └── createClobClient.ts
├── utils/
│   ├── fetchData.ts      # Polymarket API calls
│   ├── postOrder.ts      # Order submission
│   ├── logger.ts         # Logging utilities
│   └── healthCheck.ts    # System health checks
├── models/
│   └── userHistory.ts    # Trade history schema
└── scripts/              # Utility scripts (see above)
```

### Data Flow

1. **tradeMonitor** polls Polymarket API for trader activity
2. New trades are detected and validated
3. **copyStrategy** calculates appropriate order size
4. **tradeExecutor** submits orders via CLOB client
5. Results are logged and stored in MongoDB

## Finding Traders to Copy

Use `npm run find-traders` to discover profitable traders. Look for:

- **Win rate** > 55%
- **Consistent activity** (not just one lucky trade)
- **Reasonable position sizes** (relative to your capital)
- **Diverse markets** (not all-in on single events)

You can also browse Polymarket's leaderboards and use trader addresses from there.

### Multiple Traders

Copy multiple traders by comma-separating addresses:

```env
USER_ADDRESSES=0xTrader1...,0xTrader2...,0xTrader3...
```

Or use JSON array format:

```env
USER_ADDRESSES=["0xTrader1...", "0xTrader2...", "0xTrader3..."]
```

## Troubleshooting

### Common Issues

**"Health check failed"**
- Verify RPC_URL is valid and has Polygon access
- Check MongoDB connection string
- Ensure PRIVATE_KEY matches PROXY_WALLET

**"Order below minimum"**
- Increase COPY_SIZE or reduce MIN_ORDER_SIZE_USD
- Use FIXED strategy for predictable minimums

**"Insufficient balance"**
- Deposit USDC to your wallet on Polygon
- Check you have MATIC for gas fees

**"Allowance too low"**
- Run `npm run set-token-allowance`

### Log Files

Logs are stored in the `logs/` directory:
- `combined.log` - All logs
- `error.log` - Errors only

## Security Best Practices

1. **Use a dedicated wallet** - Never use your main wallet
2. **Limit initial funding** - Start with small amounts
3. **Set conservative limits** - Use MAX_ORDER_SIZE_USD
4. **Test in preview mode** - Always test before going live
5. **Monitor regularly** - Check positions daily
6. **Keep .env private** - Never commit to git
7. **Use reputable RPC** - Infura, Alchemy, or QuickNode

## API Endpoints Used

The bot only communicates with official Polymarket services:
- `https://data-api.polymarket.com` - Trader activity & positions
- `https://clob.polymarket.com` - Order book & trading
- Your configured RPC - Blockchain transactions

## License

ISC
