# General Info
### Base Websocket URI
```
"wss://stream3.bullx.io/app/prowess-frail-sensitive?protocol=7&client=js&version=8.4.0-rc2&flash=false"
```
### Required headers
```
headers = {
    "Host": "stream3.bullx.io",
    "Pragma": "no-cache",
    "Cache-Control": "no-cache",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36",
    "Origin": "https://neo.bullx.io",
    "Accept-Encoding": "gzip, deflate, br, zstd",
    "Accept-Language": "de-DE,de;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6,zh;q=0.5,zh-TW;q=0.4,zh-HK;q=0.3"
}
```
<br></br>
# Decoding encoded/compressed responses
```
def shit_decode(data_str): # this works on some (like chart/swaps)
    data_str = data_str.replace('[', '').replace(']', '').replace(' ', '')
    byte_data = bytes(map(int, data_str.split(",")))
    try:
        decompressed_data = zlib.decompress(byte_data, wbits=15 + 32)
        return decompressed_data.decode('utf-8')
    except zlib.error as e:
        #print(f"Error decompressing data: {e}")
        return None

def decode_data(data_str): # for where the above doesn't work (like liquidty pools) use msgpack
    data_str = str(data_str)
    
    dater = shit_decode(data_str)
    if dater:
        return dater

    data_str = data_str.strip('[]').replace(' ', '')
    byte_data = bytes(map(int, data_str.split(',')))
    
    try:
        unpacked_data = msgpack.unpackb(byte_data)
        return unpacked_data
    except Exception as e:
        print(f"MessagePack decoding error: {e}")
        return None
```
<br></br>

# Subscribing to token update
### Request json
```
{"event":"pusher:subscribe","data":{"auth":"","channel":"token_updates_TOKENADDRESSHERE_1399811149"}}
```
### Getting response 
```
if "token_updates" in j['event']:
    inner_data = json.loads(j['data'])
```
### Response json
```
{
    "address": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
    "chainId": 1399811149,
    "creationBlockNumber": 300283506,
    "creationBlockTimestamp": 1731114410,
    "decimals": 6,
    "deployerAddress": "E7icbbYsWHsZ4LmSnpLQCXXvEiraDjsUhBemHzayZSfy",
    "description": null,
    "goplusStats": null,
    "holdersCount": "344",
    "id": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump_1399811149",
    "isVerifiedSourceCode": null,
    "limits": {
    },
    "links": {
        "twitter": "x.com/kleavittnh",
        "website": "https://en.wikipedia.org/wiki/Karoline_Leavitt"
    },
    "liquidityPools": {
    },
    "logo": "https://ipfs.io/ipfs/QmbG5scUzEAt9XP5Sk1UcLumsCqoUxfLn5aTDGamyozMQS",
    "name": "Karoline Leavitt",
    "origin": "PUMP",
    "ownerAddress": null,
    "riskAnalysis": {
        "freezeAuthority": null,
        "isMutable": false,
        "mintAuthority": null,
        "ownerBalance": "0",
        "updateAuthority": "TSLvdd1pWpHVjahSpsvCXUbgwsL3JAcvokwaKt1eokM"
    },
    "riskStatus": "SAFE",
    "searchGraphData": null,
    "symbol": "Karoline",
    "syncedAt": "2025-01-29T03:21:39.946Z",
    "top10HoldersSupply": null,
    "totalSupply": "999999095808852",
    "version": "1",
    "website": null
}
```
<br></br>
# Subscribing to token swap events
### Request json
```
{"event":"pusher:subscribe","data":{"auth":"","channel":"blockWiseSwaps_TOKENADDRESSHERE_So11111111111111111111111111111111111111112"}}
```
### Getting response
```
if "blockWiseSwaps" in j['event']:
    print(decode_data(j['data']))
```                                
### Response json
```
{
    "blockNumber": "317050116",
    "data": [
        {
            "amountIn": "51759515938",
            "amountOut": "14987275",
            "amountUSD": 3.448871723,
            "baseTokenLpPriceNative": null,
            "baseTokenLpPriceUSD": null,
            "blockNumber": "317050116",
            "blockTimestamp": 1738121222,
            "chainId": 1399811149,
            "contractInteractedWith": "9RYJ3qr5eU5xAooqVcbmdeusjcViL5Nkiq7Gske3tiKq",
            "logIndex": 5000000,
            "makerData": {
                "address": "AtfFrg6QopYk6xfxGj2YwKEo81BhmvWFwBAveTMC6uaP",
                "chainId": 1399811149
            },
            "nativeBalance": "33076625",
            "poolAddress": "94B3YUycfb3J2yhVFcpg8hGx1M6xwryha19PQbgVNMmU",
            "protocol": "RAYDIUM",
            "suspectedBaseTokenPrice": "0.00000028955593437065",
            "suspectedBaseTokenPriceUSD": "0.00006663261161737336",
            "timestamp": 1738121222,
            "tokenIn": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
            "tokenOut": "So11111111111111111111111111111111111111112",
            "transactionHash": "deagma1fmwQwpihPi7yypnJKp87A1KaS6H9AddWWVXo565YDQYXrLNcv83RHi7sPFM2m8BKrLQuwVJfCwuyXGQ4",
            "transactionIndex": 0
        }
    ],
    "pairId": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump_So11111111111111111111111111111111111111112",
    "syncedAt": "2025-01-29T03:27:02.721Z"
}
```
<br></br>
# Subscribing to new token pairs
### Request json
```
{"event":"pusher:subscribe","data":{"auth":"","channel":"new-pairsv2-1399811149"}}
```

### Raw response
```
{
    "channel": "new-pairsv2-1399811149",
    "data": "[{\"a\":\"UD5sykck1sNdRfbC3R6N24eKEpfMmyCtAEyEpdnpump\",\"b\":\"FLOWER\",\"d\":0,\"e\":1738120596,\"f\":0,\"g\":0,\"h\":0,\"i\":0,\"j\":0,\"k\":[],\"l\":\"0.00000002795899347623\",\"m\":0,\"n\":1399811149,\"o\":\"0.00000002795899347623\",\"p\":0,\"q\":0,\"r\":\"SAFE\",\"s\":1738120596,\"t\":0,\"u\":[true,true,true,true],\"v\":6444,\"w\":{\"website\":\"https://flower.ai\",\"twitter\":\"https://x.com/flwrlabs\"},\"x\":null,\"y\":6,\"z\":\"PUMP\",\"aa\":\"Flower AI\",\"ab\":\"PUMP\",\"ac\":0,\"ad\":6,\"ae\":false,\"af\":false,\"ag\":false,\"ah\":0,\"ai\":null,\"aj\":0,\"ak\":0,\"al\":0,\"am\":0,\"an\":null,\"ao\":null,\"ap\":null,\"aq\":null,\"ar\":null,\"at\":null,\"ver\":2}]",
    "event": "new-pairsv2-1399811149"
}
```

## Clearly not usable as it is, so some transformation is needed (to get json)

```
def remap_token_data(compressed_data):
    data = json.loads(compressed_data['data'])[0]
    try:
        return {
            'image': f"image_url_{data['n']}_{data['a']}",
            'address': data['a'],
            'symbol': data['b'],
            'totalVol': data['d'],
            'creationTimestamp': data['e'] or data['s'],
            'holders': data['f'],
            'baseLiquidity': data['g'],
            'liquidityUSD': data['g'],
            'totalTxs': data['h'],
            'buys': data['i'],
            'sells': data['j'],
            'intervalData': [{'iTotalTxs': x[0], 'iPriceUSD': float(x[1])} for x in (data.get('k', []) or [])],
            'oldPriceEth': float(data.get('m', 0)),
            'chainId': data['n'],
            'priceEth': data['o'],
            'buyTax': data['p'],
            'sellTax': data['q'],
            'riskStatus': data['r'],
            'poolCreationBlockTimestamp': data['s'] or data['e'],
            'initialLiquidityETH': data['t'],
            'auditResults': data['u'],
            'marketCap': data['v'],
            'marketCapUSD': data['v'],
            'links': data['w'],
            'poolOpenTime': data['x'],
            'decimals': data['y'],
            'protocol': data['z'],
            'name': data['aa'],
            'origin': data['ab'],
            'bondingCurveProgress': data['ac'],
            'devHoldingSupplyPerc': data['ad'],
            'isDevRug': data['ae'],
            'isDevNew': data['af'],
            'isRenownedDev': data['ag'],
            'top10HoldersSupplyPerc': data['ai'],
            'sniperWalletsCount': data['aj'],
            'insiderWalletsCount': data['ak'],
            'botsTotalCount': data['al'],
            'insiderWalletsSupplyPerc': data['am'],
            'deployerAddress': data['an'],
            'description': data['ao'],
            'priceChange1mPerc': data['ap'],
            'priceChange5mPerc': data['aq'],
            'priceChange10mPerc': data['ar'],
            'chart': data['at']
        }
    except:
        print('Error remapping token data')
        return None
```

## Now we can get usable data
```
{
    "address": "UD5sykck1sNdRfbC3R6N24eKEpfMmyCtAEyEpdnpump",
    "auditResults": [
        true,
        true,
        true,
        true
    ],
    "baseLiquidity": 0,
    "bondingCurveProgress": 0,
    "botsTotalCount": 0,
    "buys": 0,
    "buyTax": 0,
    "chainId": 1399811149,
    "chart": null,
    "creationTimestamp": 1738120596,
    "decimals": 6,
    "deployerAddress": null,
    "description": null,
    "devHoldingSupplyPerc": 6,
    "holders": 0,
    "image": "image_url_1399811149_UD5sykck1sNdRfbC3R6N24eKEpfMmyCtAEyEpdnpump",
    "initialLiquidityETH": 0,
    "insiderWalletsCount": 0,
    "insiderWalletsSupplyPerc": 0,
    "intervalData": [
    ],
    "isDevNew": false,
    "isDevRug": false,
    "isRenownedDev": false,
    "links": {
        "twitter": "https://x.com/flwrlabs",
        "website": "https://flower.ai"
    },
    "liquidityUSD": 0,
    "marketCap": 6444,
    "marketCapUSD": 6444,
    "name": "Flower AI",
    "oldPriceEth": 0.0,
    "origin": "PUMP",
    "poolCreationBlockTimestamp": 1738120596,
    "poolOpenTime": null,
    "priceChange10mPerc": null,
    "priceChange1mPerc": null,
    "priceChange5mPerc": null,
    "priceEth": "0.00000002795899347623",
    "protocol": "PUMP",
    "riskStatus": "SAFE",
    "sells": 0,
    "sellTax": 0,
    "sniperWalletsCount": 0,
    "symbol": "FLOWER",
    "top10HoldersSupplyPerc": null,
    "totalTxs": 0,
    "totalVol": 0
}
```

<br></br>
# Subscribing to Liquidity Pool Events

### Request json
```
{"event":"pusher:subscribe","data":{"auth":"","channel":"liquidityPoolsV2_TOKENADDRESSHERE_So11111111111111111111111111111111111111112_1399811149"}}
```
### Getting response
```
if "liquidityPoolsV2" in j['event']:
    try:
        inner_data = json.loads(j['data'])
        if "data" in inner_data and inner_data["type"] == "Buffer":
            print("=======================")
            print(decode_data(inner_data['data']))
            print("=======================")
    except Exception as e:
        print(f"Error decoding data: {e}")
```
### Response json
```
{
    "blockUpdatedAt": 317048366,
    "chainId": 1399811149,
    "factoryAddress": "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8",
    "id": "94B3YUycfb3J2yhVFcpg8hGx1M6xwryha19PQbgVNMmU_1399811149",
    "pairId": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump_So11111111111111111111111111111111111111112",
    "poolAddress": "94B3YUycfb3J2yhVFcpg8hGx1M6xwryha19PQbgVNMmU",
    "protocol": "RAYDIUM",
    "protocolData": {
        "amountWaveRatio": "5000000",
        "baseDecimal": 9,
        "baseInfo": {
            "isNative": true,
            "mint": "So11111111111111111111111111111111111111112",
            "owner": "5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1",
            "rentExemptReserve": {
                "amount": "2039280",
                "decimals": 9,
                "uiAmount": 0.00203928,
                "uiAmountString": "0.00203928"
            },
            "state": "initialized",
            "tokenAmount": {
                "amount": "59641293545",
                "decimals": 9,
                "uiAmount": 59.641293545,
                "uiAmountString": "59.641293545"
            }
        },
        "baseLotSize": "10000000",
        "baseMint": "So11111111111111111111111111111111111111112",
        "baseNeedTakePnl": "0",
        "baseTotalPnl": "1576515632",
        "baseVault": "54zo6TguGz4ktpy4cEgsFK2choSAin3AJVq3E7ttXAJ3",
        "depth": "3",
        "id": "94B3YUycfb3J2yhVFcpg8hGx1M6xwryha19PQbgVNMmU",
        "lpMint": "HruHPbz4mR6xctZCQwbUgQbTvwutE2e1ZF62rb1Q9FmS",
        "lpMintInfo": {
            "decimals": 9,
            "freezeAuthority": null,
            "isInitialized": true,
            "mintAuthority": "5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1",
            "supply": "0"
        },
        "lpReserve": "4043044540664",
        "lpVault": "11111111111111111111111111111111",
        "marketId": "CmgZcg9woa7ps3snkjAKJsDSMRcvnyGqN3VvTxMqatgw",
        "marketInfo": {
            "asks": "8pbBMrT4iMgvHNrcPQiQdiUFknyyrp7quXbAuSVqS2uF",
            "baseDepositsTotal": "0",
            "baseFeesAccrued": "0",
            "baseLotSize": "10000000",
            "baseMint": "So11111111111111111111111111111111111111112",
            "baseVault": "3tZE8YeAYMv5GvRHefrAyQ451zJSzpPhWMEdNd3xWVMA",
            "bids": "3FxrcjKUju9jEcro99b8aZ7V6gTvkac4tx3ZgCHqw7ht",
            "eventQueue": "GLvUmP1PQE7UWnYgtTzxPRVaepLuEAhcBvrVYz9MmpH5",
            "feeRateBps": "0",
            "ownAddress": "CmgZcg9woa7ps3snkjAKJsDSMRcvnyGqN3VvTxMqatgw",
            "quoteDepositsTotal": "0",
            "quoteDustThreshold": "100",
            "quoteFeesAccrued": "0",
            "quoteLotSize": "100",
            "quoteMint": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
            "quoteVault": "Cv56XzbBg1TxSXpXZKDg6gURZTWuYxCAEzzWzG59whYW",
            "referrerRebatesAccrued": "0",
            "requestQueue": "Y42jDt2kcpnETfQ37axmT5KPvBAj4tkR9a2t4tSykJ9",
            "vaultSignerNonce": "0"
        },
        "marketProgramId": "srmqPvymJeFKQ4zGQed1GFppgkRHL9kaELCbyksJtPX",
        "maxOrder": "7",
        "maxPriceMultiplier": "1000000000",
        "minPriceMultiplier": "1",
        "minSeparateDenominator": "10000",
        "minSeparateNumerator": "5",
        "minSize": "10000000",
        "nonce": "254",
        "openOrders": "EBW6UkF4sJMSWhnmn5ws8yo7UpijLB6VcCDV2Lj2RhKX",
        "orderbookToInitTime": "0",
        "owner": "GThUX1Atko4tqhN2NaiTazWSeFWMuiUvfFnyJyUghFMJ",
        "padding": [
            "0",
            "733",
            "0"
        ],
        "pnlDenominator": "100",
        "pnlNumerator": "12",
        "poolAccountOwner": "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8",
        "poolOpenTime": "0",
        "price0": "4782015.29001122806221246719",
        "price1": "0.00000020911685541634",
        "punishCoinAmount": "0",
        "punishPcAmount": "0",
        "quoteDecimal": 6,
        "quoteInfo": {
            "isNative": false,
            "mint": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
            "owner": "5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1",
            "state": "initialized",
            "tokenAmount": {
                "amount": "285205577648238",
                "decimals": 6,
                "uiAmount": 285205577.648238,
                "uiAmountString": "285205577.648238"
            }
        },
        "quoteLotSize": "10000000",
        "quoteMint": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
        "quoteNeedTakePnl": "0",
        "quoteTotalPnl": "4333975935619",
        "quoteVault": "CpZ5WoCitisbLZRUh2SKakR3U2WU6jK8ubNYywspqcYz",
        "reserve0": "59641293545",
        "reserve1": "285205577648238",
        "resetFlag": "0",
        "state": "1",
        "status": "6",
        "swapBase2QuoteFee": "5952056643947",
        "swapBaseInAmount": "627755343356",
        "swapBaseOutAmount": "646930228776",
        "swapFeeDenominator": "10000",
        "swapFeeNumerator": "25",
        "swapQuote2BaseFee": "1569388452",
        "swapQuoteInAmount": "2380822657358478",
        "swapQuoteOutAmount": "2301997002597967",
        "systemDecimalValue": "1000000000",
        "targetOrders": "6z65dQFYtSrCZtoHAzmi1vd4pDaQFo3QFgvqZ4VHVFpH",
        "token0Address": "So11111111111111111111111111111111111111112",
        "token1Address": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
        "tradeFeeDenominator": "10000",
        "tradeFeeNumerator": "25",
        "volMaxCutRatio": "500",
        "withdrawQueue": "11111111111111111111111111111111"
    },
    "suspectedBaseToken": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
    "suspectedBaseTokenPrice": "0.00000020911685541634",
    "suspectedBaseTokenReserves": "285205577648238",
    "suspectedQuoteToken": "So11111111111111111111111111111111111111112",
    "suspectedQuoteTokenPrice": "4782015.29001122806221246719",
    "suspectedQuoteTokenReserves": "59641293545",
    "syncedAt": "2025-01-29T03:15:26.615Z",
    "token0Address": "So11111111111111111111111111111111111111112",
    "token0Decimals": 9,
    "token1Address": "Bqck6PJESDgdK9sjJ93q1DC6yExBAPUZSuxYEuSGpump",
    "token1Decimals": 6
}
```

<br></br>
# Subscribe to chart data
### Request json
```
{"event": "pusher:subscribe","data":{"auth":"","channel":"charts-all-v3"}}
```
### Getting response
```
if "charts-all-v3" in j['event']:
    chart_data = decode_data(j['data'])
```
### Response json
```
[
    {
        "address": "HiHy1Vgpen8yVQbcP6MJYjaw9RMt16YdcqnbUZEapump",
        "chainId": 1399811149,
        "chart": {
            "c": [
                7341
            ],
            "h": [
                7341
            ],
            "l": [
                6474
            ],
            "o": [
                6474
            ],
            "t": [
                1738117530
            ],
            "v": [
                467
            ]
        },
        "id": "HiHy1Vgpen8yVQbcP6MJYjaw9RMt16YdcqnbUZEapump_1399811149",
        "priceChange10mPerc": 0,
        "priceChange1mPerc": 0,
        "priceChange5mPerc": 0,
        "syncedAt": "2025-01-29T02:25:37.450Z",
        "volumeUSD10m": 467,
        "volumeUSD1m": 467,
        "volumeUSD5m": 466.794606328875
    },
    {
        "address": "6jvFs1MFVmvDeYJztxxyQit4QZuJVf7nRKjJNZnHpump",
        "chainId": 1399811149,
        "chart": {
            "c": [
                10043,
                10345,
                9870,
                6778
            ],
            "h": [
                10043,
                10345,
                10345,
                9870
            ],
            "l": [
                6876,
                10043,
                9870,
                6642
            ],
            "o": [
                6876,
                10043,
                10345,
                9870
            ],
            "t": [
                1738117440,
                1738117455,
                1738117470,
                1738117485
            ],
            "v": [
                1986,
                279,
                130,
                1740
            ]
        },
        "id": "6jvFs1MFVmvDeYJztxxyQit4QZuJVf7nRKjJNZnHpump_1399811149",
        "priceChange10mPerc": -4,
        "priceChange1mPerc": -4,
        "priceChange5mPerc": -4,
        "syncedAt": "2025-01-29T02:25:37.863Z",
        "volumeUSD10m": 4135,
        "volumeUSD1m": 4135,
        "volumeUSD5m": 4134.7620134811
    },
    {
        "address": "DDZmr7WtNYbjzjfx9ydwucbY6gZnBtc7B3wobUvspump",
        "chainId": 1399811149,
        "chart": {
            "c": [
                14183,
                13869,
                10273,
                11349,
                12065,
                13079,
                14264,
                10387,
                7936,
                6465
            ],
            "h": [
                19365,
                14183,
                13869,
                14026,
                14026,
                13079,
                14264,
                14264,
                10387,
                7936
            ],
            "l": [
                7930,
                13869,
                10254,
                10273,
                11349,
                12065,
                13079,
                10387,
                7936,
                6465
            ],
            "o": [
                7930,
                14183,
                13869,
                10273,
                11349,
                12065,
                13079,
                14264,
                10387,
                7936
            ],
            "t": [
                1738117125,
                1738117140,
                1738117155,
                1738117170,
                1738117185,
                1738117215,
                1738117260,
                1738117290,
                1738117305,
                1738117380
            ],
            "v": [
                7732,
                226,
                452,
                452,
                113,
                678,
                113,
                1809,
                1587,
                18
            ]
        },
        "id": "DDZmr7WtNYbjzjfx9ydwucbY6gZnBtc7B3wobUvspump_1399811149",
        "priceChange10mPerc": -19,
        "priceChange1mPerc": -19,
        "priceChange5mPerc": -19,
        "syncedAt": "2025-01-29T02:25:38.074Z",
        "volumeUSD10m": 13180,
        "volumeUSD1m": 18,
        "volumeUSD5m": 3518.7102488728
    }
]
```
