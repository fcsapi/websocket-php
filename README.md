# FCSAPI Realtime PHP

**PHP** Real-time WebSocket client library for **Forex**, **Cryptocurrency**, and **Stock** market data from [FCS API](https://fcsapi.com).

This library provides PHP integration with WebSocket for live market data streaming. Uses the JavaScript client library for browser-based real-time updates.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PHP Version](https://img.shields.io/badge/PHP-%3E%3D7.4-blue.svg)](https://php.net)
[![Packagist](https://img.shields.io/packagist/v/fcsapi/websocket.svg)](https://packagist.org/packages/fcsapi/websocket)

## Features

- **Real-time WebSocket** - Live price updates via WebSocket connection
- **Multi-Market Support** - Forex, Crypto, and Stock data in one library
- **PHP Backend Ready** - Easy integration with PHP applications
- **Auto-Reconnect** - Handles WebSocket connection drops automatically
- **Tab Visibility** - Smart disconnect when browser tab is hidden (saves bandwidth)
- **Heartbeat** - Built-in WebSocket keep-alive mechanism

## Demo

Use demo API key for testing: `fcs_socket_demo`

## Installation

### Composer (Recommended)
```bash
composer require fcsapi/websocket
```

### Manual Installation
1. Download or clone this repository
2. Include the JavaScript library in your PHP views

```php
<!-- Include JS library from CDN -->
<script src="https://cdn.jsdelivr.net/gh/fcsapi/websocket-php/fcs-client-lib.js"></script>

<!-- Or local file -->
<script src="<?= $baseUrl ?>/fcs-client-lib.js"></script>
```

## Quick Start

### Basic PHP Example
```php
<?php
$apiKey = 'YOUR_API_KEY'; // Or use 'fcs_socket_demo' for testing
$symbol = 'BINANCE:BTCUSDT';
$timeframe = '1D';
?>
<!DOCTYPE html>
<html>
<head>
    <title>FCS-API WebSocket Example</title>
    <script src="https://cdn.jsdelivr.net/gh/fcsapi/websocket-php/fcs-client-lib.js"></script>
</head>
<body>
    <div id="price">Loading...</div>

    <script>
        const client = new FCSClient('<?= $apiKey ?>');

        client.onmessage = (data) => {
            if (data.type === 'price' && data.prices) {
                const p = data.prices;
                if (p.mode === 'candle' || p.mode === 'initial') {
                    document.getElementById('price').innerText =
                        `${data.symbol}: $${p.c} (O:${p.o} H:${p.h} L:${p.l})`;
                }
            }
        };

        client.connect().then(() => {
            client.join('<?= $symbol ?>', '<?= $timeframe ?>');
        });
    </script>
</body>
</html>
```

### Laravel Blade Example
```php
{{-- resources/views/realtime.blade.php --}}
@extends('layouts.app')

@section('scripts')
<script src="https://cdn.jsdelivr.net/gh/fcsapi/websocket-php/fcs-client-lib.js"></script>
@endsection

@section('content')
<div id="forex-price">Loading Forex...</div>
<div id="crypto-price">Loading Crypto...</div>

<script>
    const client = new FCSClient('{{ config("services.fcs.api_key") }}');

    client.onmessage = (data) => {
        if (data.type === 'price' && data.prices) {
            const p = data.prices;
            if (p.mode === 'candle' || p.mode === 'initial') {
                if (data.symbol.startsWith('FX:')) {
                    document.getElementById('forex-price').innerText =
                        `${data.symbol}: ${p.c}`;
                } else {
                    document.getElementById('crypto-price').innerText =
                        `${data.symbol}: $${p.c}`;
                }
            }
        }
    };

    client.connect().then(() => {
        client.join('FX:EURUSD', '1D');
        client.join('BINANCE:BTCUSDT', '1D');
    });
</script>
@endsection
```

### CodeIgniter Example
```php
<?php
// app/Controllers/Realtime.php
namespace App\Controllers;

class Realtime extends BaseController
{
    public function index()
    {
        $data = [
            'apiKey' => getenv('FCS_API_KEY') ?: 'fcs_socket_demo',
            'symbols' => ['FX:EURUSD', 'FX:GBPUSD', 'BINANCE:BTCUSDT']
        ];
        return view('realtime', $data);
    }
}
```

```php
<!-- app/Views/realtime.php -->
<!DOCTYPE html>
<html>
<head>
    <title>Real-time Prices</title>
    <script src="https://cdn.jsdelivr.net/gh/fcsapi/websocket-php/fcs-client-lib.js"></script>
</head>
<body>
    <table id="prices">
        <thead>
            <tr>
                <th>Symbol</th>
                <th>Open</th>
                <th>High</th>
                <th>Low</th>
                <th>Close</th>
                <th>Volume</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <script>
        const client = new FCSClient('<?= esc($apiKey) ?>');
        const symbols = <?= json_encode($symbols) ?>;

        client.onmessage = (data) => {
            if (data.type === 'price' && data.prices) {
                updatePriceRow(data.symbol, data.prices);
            }
        };

        function updatePriceRow(symbol, p) {
            if (p.mode !== 'candle' && p.mode !== 'initial') return;

            let row = document.getElementById('row-' + symbol.replace(':', '-'));
            if (!row) {
                row = document.createElement('tr');
                row.id = 'row-' + symbol.replace(':', '-');
                document.querySelector('#prices tbody').appendChild(row);
            }
            row.innerHTML = `
                <td>${symbol}</td>
                <td>${p.o}</td>
                <td>${p.h}</td>
                <td>${p.l}</td>
                <td>${p.c}</td>
                <td>${p.v || '-'}</td>
            `;
        }

        client.connect().then(() => {
            symbols.forEach(s => client.join(s, '1D'));
        });
    </script>
</body>
</html>
```

## API Reference

### JavaScript Client (Browser-side)

#### Constructor
```javascript
const client = new FCSClient(apiKey, url);
```
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `apiKey` | string | required | Your FCS API key |
| `url` | string | `wss://ws-v4.fcsapi.com/ws` | WebSocket server URL (optional) |

### Methods

#### `connect()`
Connects to the WebSocket server. Returns a Promise.
```javascript
client.connect().then(() => {
    console.log('Connected!');
}).catch(err => {
    console.error('Connection failed:', err);
});
```

#### `disconnect()`
Manually closes the connection.
```javascript
client.disconnect();
```

#### `join(symbol, timeframe)`
Subscribe to a symbol for real-time updates.
```javascript
// Forex
client.join('FX:EURUSD', '1m');

// Crypto
client.join('BINANCE:BTCUSDT', '1m');

// Stock
client.join('NASDAQ:AAPL', '1m');
```

#### `leave(symbol, timeframe)`
Unsubscribe from a symbol.
```javascript
client.leave('FX:EURUSD', '1m');
```

#### `removeAll()`
Unsubscribe from all symbols.
```javascript
client.removeAll();
```

### Event Callbacks

| Callback | Description |
|----------|-------------|
| `onconnected` | Fired when connection is established |
| `onmessage` | Fired when data is received |
| `onclose` | Fired when connection is closed |
| `onerror` | Fired when an error occurs |
| `onreconnect` | Fired when reconnection is successful |

```javascript
client.onconnected = () => console.log('Connected!');
client.onmessage = (data) => console.log('Data:', data);
client.onclose = (event) => console.log('Closed:', event.code);
client.onerror = (err) => console.error('Error:', err);
client.onreconnect = () => console.log('Reconnected!');
```

### Configuration Options

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `reconnectDelay` | number | 3000 | Delay (ms) before reconnection attempt |
| `reconnectlimit` | number | 5 | Maximum reconnection attempts |
| `focusTimeout` | number | 3 | Minutes before disconnect when tab is hidden (0 = never) |

```javascript
const client = new FCSClient('YOUR_API_KEY');
client.reconnectDelay = 5000;  // 5 seconds
client.reconnectlimit = 10;    // 10 attempts
client.focusTimeout = 5;       // 5 minutes
```

### Message Data Format

The WebSocket sends different message types:

#### 1. Join Confirmation
```javascript
{
    "type": "message",
    "success": true,
    "message": "Successfully joined room: BINANCE:BTCUSDT_1D_0s",
    "room": "BINANCE:BTCUSDT_1D_0s",
    "symbol": "BTCUSDT",
    "timeframe": "1D",
    "short": "joined_room"
}
```

#### 2. Profile Data (One-time on join)
```javascript
{
    "type": "price",
    "symbol": "BINANCE:BTCUSDT",
    "timeframe": "1D",
    "prices": {
        "mode": "profile",
        "profile": {
            "current_session": "market",
            "timezone": "Etc/UTC",
            "pro_name": "BINANCE:BTCUSDT",
            "update_mode": "streaming"
        }
    }
}
```

#### 3. Initial Candle Data (One-time on join)
```javascript
{
    "type": "price",
    "symbol": "BINANCE:BTCUSDT",
    "timeframe": "1D",
    "prices": {
        "mode": "initial",
        "t": 1766361600,      // Timestamp (Unix seconds)
        "o": 88658.87,        // Open price
        "h": 90588.23,        // High price
        "l": 87900,           // Low price
        "c": 89962.61,        // Close price
        "v": 0.247            // Volume
    }
}
```

#### 4. Live Candle Updates (Continuous)
Primary update mode - contains all price data including OHLCV and Ask/Bid.
```javascript
{
    "type": "price",
    "symbol": "BINANCE:BTCUSDT",
    "timeframe": "1D",
    "prices": {
        "mode": "candle",
        "t": 1766361600,      // Timestamp
        "o": 88658.87,        // Open
        "h": 90588.23,        // High
        "l": 87900,           // Low
        "c": 89962.61,        // Close
        "v": 8192.70,         // Volume
        "a": 89962.62,        // Ask price
        "b": 89962.61         // Bid price
    }
}
```

#### 5. Ask/Bid Updates
Sent when only Ask/Bid data changes. All values in this message are updated.
```javascript
{
    "type": "price",
    "symbol": "BINANCE:BTCUSDT",
    "timeframe": "1D",
    "prices": {
        "mode": "askbid",
        "update": 1766411426, // Update timestamp
        "c": 89962.61,        // Close price
        "a": 89962.62,        // Ask price
        "b": 89962.61,        // Bid price
        "t": 1766361600       // Candle timestamp
    }
}
```

### Handling Different Price Modes

```javascript
client.onmessage = (data) => {
    if (data.type === 'price' && data.prices) {
        const p = data.prices;
        const symbol = data.symbol;

        switch (p.mode) {
            case 'profile':
                console.log(`${symbol} Profile loaded`);
                break;

            case 'initial':
            case 'candle':
                console.log(`${symbol}: O=${p.o} H=${p.h} L=${p.l} C=${p.c} V=${p.v} Ask=${p.a} Bid=${p.b}`);
                break;

            case 'askbid':
                console.log(`${symbol}: Ask=${p.a} Bid=${p.b}`);
                break;
        }
    }
};
```

## Symbol Format

Symbols must include an exchange prefix:

| Market | Format | Examples |
|--------|--------|----------|
| Forex | `FX:PAIR` | `FX:EURUSD`, `FX:GBPUSD`, `FX:USDJPY` |
| Crypto | `EXCHANGE:PAIR` | `BINANCE:BTCUSDT`, `BINANCE:ETHUSDT` |
| Stock | `EXCHANGE:SYMBOL` | `NASDAQ:AAPL`, `NYSE:TSLA` |

## Timeframes

| Timeframe | Description |
|-----------|-------------|
| `1` | 1 minute |
| `5` | 5 minutes |
| `15` | 15 minutes |
| `30` | 30 minutes |
| `1H` | 1 hour |
| `4H` | 4 hours |
| `1D` | 1 day |
| `1W` | 1 week |
| `1M` | 1 month |

## Examples

Check the `/examples` folder for complete working demos:

- [forex-example.php](examples/forex-example.php) - Real-time Forex prices
- [crypto-example.php](examples/crypto-example.php) - Real-time Cryptocurrency prices
- [stock-example.php](examples/stock-example.php) - Real-time Stock prices

## PHP Framework Integration

### Laravel
Add to your `.env`:
```
FCS_API_KEY=your_api_key_here
```

Add to `config/services.php`:
```php
'fcs' => [
    'api_key' => env('FCS_API_KEY', 'fcs_socket_demo'),
],
```

### CodeIgniter 4
Add to your `.env`:
```
FCS_API_KEY=your_api_key_here
```

### Symfony
Add to your `.env`:
```
FCS_API_KEY=your_api_key_here
```

## Browser Tab Visibility

The library automatically handles browser tab visibility to save bandwidth:

- When tab is hidden for more than 3 minutes (configurable), connection is closed
- When tab becomes visible again, connection is automatically restored
- All subscriptions are automatically rejoined after reconnection

Set `focusTimeout = 0` to disable this feature:
```javascript
client.focusTimeout = 0; // Never disconnect when tab hidden
```

## Error Handling

```javascript
client.onerror = (err) => {
    console.error('WebSocket error:', err);
};

client.onclose = (event) => {
    if (event.code !== 1000) {
        console.error('Unexpected close:', event.code, event.reason);
    }
};
```

## Get API Key

1. Visit [FCS API](https://fcsapi.com)
2. Sign up for a free account
3. Get your API key from the dashboard

## Documentation

For complete API documentation, visit:
- [FCS API Documentation](https://fcsapi.com/document/stock-api)
- [WebSocket API Guide](https://fcsapi.com/document/stock-api#websocket)

## Support

- Email: support@fcsapi.com
- Website: [fcsapi.com](https://fcsapi.com)

## License

MIT License - see [LICENSE](LICENSE) file for details.
