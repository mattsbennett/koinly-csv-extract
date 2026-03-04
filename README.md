# koinly-csv-extract
Extract of ergo transactions in koinly csv format. Please keep in mind this is a first version and probably will never be 100% correct in all cases. Make sure you do a sanity check once the csv is imported into Koinly.

## Features
- Breaks up multi asset transactions so each token gets imported into Koinly
- Assigns a name to each token/nft that is not known by Koinly and makes sure it is consistent between your wallets (NULL1, NULL2, NFT1, NFT2 etc.) Look in the description for transactions for which token it is about.
- Combines trades on fe. Spectrum into one exchange transaction in Koinly
- Fetches price of tokens with value to assign a networth to transactions in Koinly
- Ergopad vesting redeems are handled according to this description: [Koinly ICO transactions](https://help.koinly.io/en/articles/3732271-how-do-i-enter-ico-transactions)
- Ergopad staking transactions are handled according to this description: [Koinly Staking](https://help.koinly.io/en/articles/4928636-staking)

## Requirements
To use this tool you need to have [Python](https://www.python.org/) installed. Once it has been tested I will generate standalone executables to make it easier for the average user.

You will need a CoinGecko API key to fetch the price of ERG. A paid (basic+) account is required since the pro API endpoint is used. Register at [CoinGecko](https://www.coingecko.com/en/api) and add your key to a `.env` file:
```
COINGECKO_API_KEY="yourapikey"
```

## Usage
1. Download the files in this repository using either git or just download (Green "Code" dropdown on top right -> "Download as zip")
2. In the folder with the downloaded files edit the file "wallets.json" with a text editor to match your needs.
3. *First use only* Install required Python packages with pip by calling this command:
```
pip install -r requirements.txt
```
4. Run the extraction, it will generate a csv for each wallet defined in wallets.json. By default it extracts transactions from block 1429491 (Jan 1 2025) onwards:
```
python main.py extract
```

If you want to extract transactions in a specific block range only (fe. between block 400000 and 450000) you can add parameters like this:
```
python main.py extract --fromheight 400000 --toheight 450000
```

## Token ID management (year-over-year)

Koinly doesn't know most Ergo tokens, so they get assigned placeholder IDs (NULL1, NULL2, NFT1, NFT2, etc.). These IDs need to be consistent across years so Koinly can track holdings correctly.

Each year's master list of token ID assignments is stored in an `Export-Final <year>.csv` file (e.g. `Export-Final 2024.csv`). When starting a new tax year:

1. Run the extraction to generate fresh wallet CSVs
2. Compare the token names in the new wallet CSVs against the previous year's `Export-Final` to find tokens that already have IDs
3. Remap existing tokens to their previous IDs, and assign new sequential IDs to any tokens not seen before
4. Write a new `Export-Final <year>.csv` containing all previous entries plus the new ones
5. Update all wallet CSVs with the remapped IDs

This ensures that if you received CRUX as `NULL45` in 2024, it stays `NULL45` in 2025.

## Updating for a new year

The following values in `main.py` are hardcoded for the current tax year and need updating each year:

- **`fetchErgoPriceHistory()`**: The `from` and `to` unix timestamps in the CoinGecko API URL (currently `from=1735689600` / `to=1767225599` for Jan 1 2025 - Dec 31 2025)
- **`extract()` default `fromHeight`**: The starting block height (currently `1429491` for Jan 1 2025)

## Price sources

- **ERG/USD**: Fetched from CoinGecko pro API (`/coins/ergo/market_chart/range`)
- **Token/ERG**: Fetched from [Crux Finance](https://cruxfinance.io) Spectrum API (`/spectrum/price`), which provides historical token prices from Spectrum DEX pools. Prices are cached per token per day to reduce API calls, with retry logic for transient errors.