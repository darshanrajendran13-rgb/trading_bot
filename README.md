# trading_bot
# Simplified Trading Bot — Binance Futures Testnet (USDT-M)

A small, structured Python CLI application for placing MARKET, LIMIT, and
STOP_LIMIT orders on Binance Futures Testnet.

## Project structure

```
trading_bot/
  bot/
    __init__.py
    client.py           # Talks to Binance. The only file that imports `binance`.
    orders.py           # Builds order params, prints request/response summaries.
    validators.py       # Validates CLI input before anything touches the network.
    logging_config.py   # One shared logger, writes to bot.log + console.
  cli.py                # argparse entry point — wires the above together.
  requirements.txt
  bot.log               # Created automatically the first time you run cli.py
  README.md
```

## Setup

1. **Create a Binance Futures Testnet account**
   Go to https://testnet.binancefuture.com, log in with a GitHub account, and
   activate the Futures Testnet.

2. **Generate API credentials**
   On the testnet site, go to the API Key section and generate a key/secret
   pair. (These are testnet-only keys — they do not work on real Binance and
   carry no real funds.)

3. **Install dependencies**
   ```bash
   cd trading_bot
   python3 -m venv venv
   source venv/bin/activate        # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

4. **Set your API credentials as environment variables**
   ```bash
   export BINANCE_API_KEY="your_testnet_api_key"
   export BINANCE_API_SECRET="your_testnet_api_secret"
   ```
   (On Windows PowerShell: `$env:BINANCE_API_KEY="..."`)

   Credentials are read from the environment on purpose — never hardcode
   keys in source code, since that's the single most common way API keys
   leak onto public GitHub repos.

## How to run

**Market order:**
```bash
python cli.py --symbol BTCUSDT --side BUY --type MARKET --quantity 0.01
```

**Limit order:**
```bash
python cli.py --symbol BTCUSDT --side SELL --type LIMIT --quantity 0.01 --price 70000
```

**Stop-limit order (bonus order type):**
```bash
python cli.py --symbol BTCUSDT --side SELL --type STOP_LIMIT \
  --quantity 0.01 --price 60000 --stop-price 60500
```

Every run prints:
- an order request summary (what's being sent)
- the order response (orderId, status, executedQty, avgPrice if available)
- a success or failure message

Every run also appends structured entries to `bot.log`, including the raw
request dict and raw response dict, plus any validation or API errors.

## Assumptions

- Only USDT-M futures pairs are supported (as required by the assignment);
  quantity/price precision rules for each symbol are enforced by Binance
  itself — if a value doesn't match the symbol's step size, the API will
  reject it and the error will be logged.
- `python-binance` is used rather than raw REST calls, since it already
  handles request signing, timestamps, and retries correctly for the
  Futures Testnet.
- Default `timeInForce` is `GTC` (Good-Til-Cancelled) for LIMIT and
  STOP_LIMIT orders; this can be overridden with `--time-in-force IOC` or
  `--time-in-force FOK`.
- The bonus feature implemented is a third order type: **STOP_LIMIT**
  (maps to Binance's `STOP` futures order type, which carries both a
  trigger `stopPrice` and an execution `price`).

## Important note on log files

The assignment asks for log files from at least one real MARKET order and
one real LIMIT order. You must generate these yourself by running the
commands above with your own testnet API keys — that's the only way the
logs reflect a genuine round-trip to Binance's testnet, which is presumably
what's being evaluated. Delete/replace `bot.log` before your first real run
so it only contains your own submitted orders.
