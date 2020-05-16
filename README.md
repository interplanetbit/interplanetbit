# InterplanetBit

InterplanetBit is the world's most advantage crypto exchange platform.

### Products

- Incredible high-speed, extremely secure, support both REST and WebSocket.
- Spots Trading System: Supports up to 100K orders/s.
- Futures Trading System: Supports up to 10K orders/s.
- Supports futures, perpetual and inverse contracts.
- Wallet System: Support cold/hot wallet and offline signature. Supports BTC/BCH/LTC/ETH/ERC20.

### Extremely high-speed Trading Engine

InterplanetBit designed and developed the world leading high-speed trading engine which supports up to 100K orders per second. This advanced trading engine is a 100% pure in-memory trading ending which is extramely fast and provides synchronzied API for order.

NOTE: the response of the synchronized API is actual result of order processing which means all trading task such as freeze, match and clearing are done. No extra async query is required.

The whole trading system does NOT need expensive high-performance database because of the benifits on high-speed memory engine. This is greatly reduced the overall cost.

The high-speed trading engine supports stateless cluster, with periodic snapshot, very low latency, high throughputs and it is extramely stable. It also supports hot upgrade while users are unawared.

### Supports ALL types of orders and fee models

InterplanetBit supports limit order, market order and:

- Stop order;
- Stop limit order;
- Trailing stop order;
- FOK (fill-or-kill) order;
- IOC (immediate-or-cancel order;
- Hidden order;
- Post-only order.

The fee sub-system also supports different fee rates for taker/maker, and supports negative fee rate for maker. It supports update fee rates by symbols, time, user groups and individual user.

### Flexible Operation

The exchange system can be modified without down-time:

- Support add new currencies and symbols.
- Support update fee settings.
- Support update system parameters like limitations of active orders.

### Cloud Deploy and Monitor

InterplanetBit supports cloud deployment. You can deploy exchange system on AWS:

- Running on Linux and OpenJDK 11 platform.
- Support AutoScaling policy on EC2.
- Support blue/green deployment.
- Support managed MySQL/Aurora and read/write separation.
- Support managed Kafka and Redis.

InterplanetBit does not need high resources. A 4u8g server can support 10K transactions per second. It also supports table sharding and no need to move data. Monitors like DataDog are fully integrated.
