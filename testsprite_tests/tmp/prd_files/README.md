# SecureBank API

A production-grade RESTful Banking Transaction API built with Spring Boot 3.x, demonstrating enterprise-level architecture and best practices.

## 🚀 Features

### Core Banking Features
- **User Management**: Registration, authentication with JWT
- **Account Management**: Create savings/current accounts, view balances, statements
- **Transaction Processing**: Deposit, withdraw, transfer with ACID compliance
- **Security**: Role-based access (USER/ADMIN), JWT authentication, audit logging
- **Business Logic**: Daily limits, minimum balance, transaction fees, fraud detection

### Technical Features
- **ACID Transactions**: Atomic transfers with pessimistic locking
- **Comprehensive Testing**: Unit tests with Mockito, integration tests with TestContainers
- **API Documentation**: Swagger/OpenAPI integration
- **Containerization**: Docker and Docker Compose setup
- **Database**: PostgreSQL with proper indexing and constraints

## 🏗️ Architecture

```
securebank-backend/
├── config/          # Security, Swagger configuration
├── controller/      # REST endpoints (Auth, Account, Transaction, Admin)
├── dto/            # Request/Response objects
├── entity/         # JPA entities (User, Account, Transaction, AuditLog)
├── enums/          # Domain enums
├── repository/     # JPA repositories
├── service/        # Business logic layer
├── exception/      # Custom exceptions
└── util/           # Utilities (Account number generator)
```

## 🛠️ Tech Stack

- **Backend**: Java 17, Spring Boot 3.2, Spring Security, Spring Data JPA
- **Database**: PostgreSQL 15
- **Security**: JWT, BCrypt, Role-based access control
- **Testing**: JUnit 5, Mockito, TestContainers
- **Documentation**: Swagger/OpenAPI
- **Containerization**: Docker, Docker Compose

## 📋 Prerequisites

- Java 17+
- Docker & Docker Compose
- Maven 3.8+

## 🚀 Quick Start

### Using Docker Compose (Recommended)

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd securebank-backend
   ```

2. **Build and run with Docker**
   ```bash
   docker-compose up --build
   ```

3. **Access the application**
   - API: http://localhost:8080
   - Swagger UI: http://localhost:8080/swagger-ui.html
   - Database: localhost:5432 (postgres/postgres)

### Manual Setup

1. **Start PostgreSQL**
   ```bash
   docker run -d --name postgres -p 5432:5432 \
     -e POSTGRES_DB=securebank \
     -e POSTGRES_USER=postgres \
     -e POSTGRES_PASSWORD=postgres \
     postgres:15
   ```

2. **Build and run**
   ```bash
   mvn clean install
   mvn spring-boot:run
   ```

## 📚 API Documentation

### Authentication Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register new user |
| POST | `/api/auth/login` | Login and get JWT token |
| GET | `/api/auth/me` | Get current user profile |

### Account Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/accounts` | Create new account |
| GET | `/api/accounts` | Get user's accounts |
| GET | `/api/accounts/{id}` | Get account details |
| GET | `/api/accounts/{id}/balance` | Get current balance |
| GET | `/api/accounts/{id}/statement` | Get account statement |
| PATCH | `/api/accounts/{id}/status` | Update account status (Admin) |

### Transaction Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/transactions/deposit` | Deposit money |
| POST | `/api/transactions/withdraw` | Withdraw money |
| POST | `/api/transactions/transfer` | Transfer between accounts |
| GET | `/api/transactions` | Get user's transactions |
| GET | `/api/transactions/{id}` | Get transaction details |

### Admin Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/users` | Get all users |
| GET | `/api/admin/accounts` | Get all accounts |
| GET | `/api/admin/transactions` | Get all transactions |
| POST | `/api/admin/accounts/{id}/freeze` | Freeze account |
| POST | `/api/admin/accounts/{id}/unfreeze` | Unfreeze account |
| GET | `/api/admin/reports/daily` | Get daily report |
| GET | `/api/admin/audit-logs` | Get audit logs |

## 🧪 Testing

### Run Unit Tests
```bash
mvn test
```

### Run Integration Tests
```bash
mvn verify
```

### Test Coverage
- Unit tests: 85%+ coverage with Mockito
- Integration tests: End-to-end with TestContainers
- API tests: Controller layer testing

## 🔒 Security Features

- **JWT Authentication**: Stateless token-based auth
- **Password Encryption**: BCrypt hashing
- **Role-based Access**: USER and ADMIN roles
- **Audit Logging**: All sensitive operations logged
- **Input Validation**: Comprehensive request validation
- **SQL Injection Protection**: JPA/Hibernate ORM

## 💾 Database Schema

### Users Table
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    role VARCHAR(20) DEFAULT 'USER',
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Accounts Table
```sql
CREATE TABLE accounts (
    id BIGSERIAL PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    account_type VARCHAR(20) NOT NULL,
    balance DECIMAL(15,2) DEFAULT 0.00,
    currency VARCHAR(3) DEFAULT 'INR',
    status VARCHAR(20) DEFAULT 'ACTIVE',
    user_id BIGINT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT positive_balance CHECK (balance >= 0)
);
```

### Transactions Table
```sql
CREATE TABLE transactions (
    id BIGSERIAL PRIMARY KEY,
    transaction_id VARCHAR(50) UNIQUE NOT NULL,
    transaction_type VARCHAR(20) NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    fee DECIMAL(10,2) DEFAULT 0.00,
    description TEXT,
    from_account_id BIGINT REFERENCES accounts(id),
    to_account_id BIGINT REFERENCES accounts(id),
    status VARCHAR(20) DEFAULT 'COMPLETED',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT positive_amount CHECK (amount > 0)
);
```

### Audit Logs Table
```sql
CREATE TABLE audit_logs (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50),
    entity_id BIGINT,
    ip_address VARCHAR(45),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 🔧 Configuration

Key configuration properties in `application.properties`:

```properties
# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/securebank
spring.datasource.username=postgres
spring.datasource.password=postgres

# JWT
jwt.secret=your-super-secret-key-for-jwt-token-generation-min-256-bits
jwt.expiration=86400000

# Transaction Limits
transaction.daily.limit=50000
transaction.minimum.balance=500
transaction.transfer.fee=10
```

## 🎯 Business Rules

- **Daily Transaction Limit**: ₹50,000 per account per day
- **Minimum Balance**: ₹500 for savings accounts
- **Transfer Fee**: ₹10 per transfer
- **Account Types**: Savings and Current
- **Account Status**: Active, Frozen, Closed
- **Transaction Types**: Deposit, Withdraw, Transfer

## 📊 Monitoring & Health Checks

- **Actuator Endpoints**: `/actuator/health`, `/actuator/info`
- **Database Health**: Automatic PostgreSQL connectivity checks
- **Audit Logs**: Complete transaction and admin action logging

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- Spring Boot team for the excellent framework
- PostgreSQL for robust database features
- TestContainers for integration testing
- Swagger for API documentation

---

**Note**: This is a demonstration project showcasing banking API development. In production, additional security measures, monitoring, and compliance features would be required.