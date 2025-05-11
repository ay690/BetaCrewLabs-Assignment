# BetaCrew Mock Exchange Client

This Node.js client connects to the BetaCrew mock exchange server over TCP, streams all order-book packets, requests any missing packets, and writes the complete, sequence-ordered data to a JSON file.

## Table of Contents

* [Prerequisites](#prerequisites)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)
* [How It Works](#how-it-works)
* [Output](#output)
* [Error Handling & Reconnection](#error-handling--reconnection)
* [License](#license)

## Prerequisites

* **Node.js** v16.17.0 or later
* A running BetaCrew mock exchange server on `localhost:3000`

## Installation

1. Clone this repository or copy the client files into your project directory.


## Configuration

The client uses the following constants at the top of `client.js`. You can adjust them if needed:

| Constant           | Description                                 | Default            |
| ------------------ | ------------------------------------------- | ------------------ |
| `HOST`             | Server hostname or IP                       | `'127.0.0.1'`      |
| `PORT`             | Server TCP port                             | `3000`             |
| `PACKET_JSON_FILE` | Name of the output JSON file                | `'orderBook.json'` |
| `PACKET_SIZE`      | Size in bytes of each binary packet (fixed) | `17`               |

## Usage

Run the main with:

```bash
node main.js
```

Run the client with:

```bash
node client.js
```

On success, you will see console logs:

```
Connected to server.
Received 15 packets from stream.
Missing sequences: 3, 7
orderBook.json created with 15 packets.
```

The output JSON file will appear in the working directory.

## How It Works

1. **Stream All Packets**: Client sends a 2-byte request (`callType = 1`) to request every packet.
2. **Parse & Collect**: Incoming TCP data is buffered; each 17-byte chunk is parsed into an object:

   ```js
   { symbol, buySellIndicator, quantity, price, packetSequence }
   ```
3. **Detect Missing**: The highest `packetSequence` is computed, and any gaps (e.g. sequences that never arrived) are tracked.
4. **Resend Requests**: For each missing sequence, a new 2-byte request (`callType = 2`, `resendSeq = N`) is sent, and the single-packet response is parsed.
5. **Merge & Sort**: All packets (initial + recovered) are combined and sorted by `packetSequence`.
6. **Write JSON**: The complete array is written to `orderBook.json`.

## Output

The JSON file is an array of objects:

```json
[
  {
    "symbol": "MSFT",
    "buySellIndicator": "B",
    "quantity": 50,
    "price": 100,
    "packetSequence": 1
  },
  ...
]
```

## Error Handling & Reconnection

* Any TCP errors or unexpected disconnects during **stream-all** will reject the promise and log an error.
* Resend requests use individual connections; failures reject that packet’s promise.
* If needed, you can wrap `requestSeq()` calls in retry logic or add exponential backoff.

## License

Distributed under the MIT License.
