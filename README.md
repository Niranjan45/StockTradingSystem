# Stock Trading Order Engine

A production-grade Spring Boot backend that simulates a simplified stock exchange.

## Features

| Feature | Details |
|---|---|
| Order Types | BUY / SELL |
| Order Statuses | OPEN → PARTIAL → FILLED \| CANCELLED |
| Matching | Price-time (FIFO) priority |
| Partial Fills | Supported |
| Concurrency | Per-stock `ReentrantLock` |
| Transactions | `@Transactional` on all mutations |
| API Docs | Swagger UI at `/swagger-ui.html` |

---

## Prerequisites

| Tool | Version |
|---|---|
| Java | 17+ |
| Maven | 3.8+ |
| MySQL | 8.0+ (or PostgreSQL 14+) |

---

## Database Setup (MySQL)

```sql
CREATE DATABASE trading_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Update credentials in `src/main/resources/application.properties`:

```properties
spring.datasource.username=YOUR_USER
spring.datasource.password=YOUR_PASSWORD
```

---

## Build & Run

```bash
# Clone / unzip the project
cd stock-trading-engine

# Build
mvn clean package -DskipTests

# Run
java -jar target/stock-trading-engine-1.0.0.jar

# Or with Maven directly
mvn spring-boot:run
```

The server starts on **http://localhost:8080**

---

## Swagger UI

Open **http://localhost:8080/swagger-ui.html**

---

## Key Endpoints

### Users
| Method | URL | Description |
|---|---|---|
| POST | `/users` | Register user |
| GET | `/users/{id}` | Get user |
| GET | `/users` | List all users |

### Stocks
| Method | URL | Description |
|---|---|---|
| POST | `/stocks` | Create stock |
| GET | `/stocks` | List all stocks |
| GET | `/stocks/{symbol}` | Get by symbol |

### Orders
| Method | URL | Description |
|---|---|---|
| POST | `/orders/buy` | Place BUY order |
| POST | `/orders/sell` | Place SELL order |
| DELETE | `/orders/{id}?userId=X` | Cancel order |
| GET | `/orders/user/{userId}` | User order history |
| GET | `/orders/orderbook/{symbol}` | Market depth |

### Trades
| Method | URL | Description |
|---|---|---|
| GET | `/trades/stock/{symbol}` | Trades for a stock |
| GET | `/trades/user/{userId}` | Trades for a user |

---

## Sample Requests

### Register User
```json
POST /users
{
  "name": "Alice",
  "email": "alice@trading.com",
  "balance": 100000.00
}
```

### Place BUY Order
```json
POST /orders/buy
{
  "userId": 1,
  "symbol": "AAPL",
  "price": 200.00,
  "quantity": 100
}
```

### Place SELL Order (will auto-match)
```json
POST /orders/sell
{
  "userId": 2,
  "symbol": "AAPL",
  "price": 200.00,
  "quantity": 100
}
```

---

## Run Tests

```bash
mvn test
```

Tests use an in-memory H2 database — no MySQL required.

---

## Project Structure

```
src/main/java/com/trading/engine/
├── StockTradingEngineApplication.java
├── config/
│   ├── DataInitializer.java      # Seeds demo stocks & users
│   └── OpenApiConfig.java        # Swagger configuration
├── controller/
│   ├── UserController.java
│   ├── StockController.java
│   ├── OrderController.java
│   └── TradeController.java
├── dto/
│   ├── request/                  # Validated request DTOs
│   └── response/                 # Response DTOs + ApiResponse wrapper
├── entity/
│   ├── User.java
│   ├── Stock.java
│   ├── Order.java
│   └── Trade.java
├── enums/
│   ├── OrderType.java
│   └── OrderStatus.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   ├── BusinessException.java
│   └── ResourceNotFoundException.java
├── repository/
│   ├── UserRepository.java
│   ├── StockRepository.java
│   ├── OrderRepository.java      # JPQL matching queries
│   └── TradeRepository.java
└── service/
    ├── impl/
    │   ├── UserServiceImpl.java
    │   ├── StockServiceImpl.java
    │   ├── OrderServiceImpl.java  # ← Matching engine here
    │   └── TradeServiceImpl.java
    ├── UserService.java
    ├── StockService.java
    ├── OrderService.java
    └── TradeService.java
```

---

## Architecture Notes

- **Matching engine** lives in `OrderServiceImpl.matchOrder()` — pure price-time priority
- **Concurrency safety**: each stock symbol has its own `ReentrantLock` (fair mode), so orders for different symbols run in parallel while same-symbol orders are serialized
- **Atomicity**: the entire place-order-and-match flow is wrapped in `@Transactional`
- **Database indexes**: added on `status`, `order_type+price+status`, `symbol`, `email` for query efficiency
