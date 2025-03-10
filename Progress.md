# Wallet Service - Progress Document

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Core Components](#core-components)
3. [Security Implementation](#security-implementation)
4. [KMS Integration](#kms-integration)
5. [Message Queue System](#message-queue-system)
6. [Transaction Processing](#transaction-processing)
7. [Monitoring & Metrics](#monitoring--metrics)
8. [Error Handling & Resilience](#error-handling--resilience)
9. [API Documentation](#api-documentation)
10. [Testing Framework](#testing-framework)
11. [Deployment Considerations](#deployment-considerations)
12. [Future Enhancements](#future-enhancements)
13. [Redis Integration](#redis-integration)

## Architecture Overview

The Wallet Service follows a modern microservice architecture with the following key design principles:

### Design Patterns
- **Hexagonal Architecture**: Clear separation between domain logic, application services, and infrastructure
- **Message-Driven Design**: Asynchronous communication via RabbitMQ
- **Circuit Breaker Pattern**: Resilience against external service failures
- **Repository Pattern**: Abstraction for data access
- **Factory Pattern**: For wallet and transaction creation
- **Caching Strategy**: Redis-based caching for performance optimization

### Technology Stack
- **Runtime**: Node.js with TypeScript
- **Queue System**: RabbitMQ
- **Security**: Google Cloud KMS
- **Metrics**: Prometheus
- **Logging**: Structured JSON logging
- **HTTP Server**: Express.js
- **Blockchain Interaction**: Solana Web3.js
- **Caching**: Redis for high-performance data caching

### Service Boundaries
The service is responsible for:
- Wallet management (creation, listing, retrieval)
- Transaction signing
- Balance queries
- Token management
- Key security
- Rate limiting and request throttling

---

## Core Components

### Application Layer
- **App.ts**: Main application bootstrapping and lifecycle management
- **ExpressServer**: HTTP API and middleware configuration
- **MessageHandler**: Processing incoming message queue requests

### Service Layer
- **WalletService**: Core business logic for wallet operations
- **SecureKMSService**: Integration with Google Cloud KMS
- **QueueService**: Message queue abstraction and management

### Infrastructure Layer
- **Constants**: System-wide configuration constants
- **Config**: Environment-specific configuration
- **Utils**: Reusable utilities (logging, monitoring, metrics)

### Type System
- Strong typing throughout the application
- Comprehensive interfaces for all domain objects
- Type declarations for external libraries

---

## Security Implementation

### Key Security Model

The Wallet Service implements a zero-trust security model where private keys are never exposed to the application:

1. **Key Generation**: When a new wallet is created, a key pair is generated in Google Cloud KMS
2. **Key Storage**: Only the public key and a reference to the KMS key are stored in the service
3. **Signing Operations**: All signing operations are performed remotely within KMS
4. **Key Access Control**: IAM policies restrict access to cryptographic operations

### Authentication & Authorization

- **User-Based Access Control**: All wallet operations are tied to a specific user ID
- **Ownership Verification**: Each operation validates the wallet belongs to the requesting user
- **Request Validation**: Comprehensive validation of all incoming requests

### Data Security

- **No Private Key Storage**: Private keys never exist in plaintext in the application
- **Secure Storage**: Wallet information is stored with appropriate access controls
- **Encryption at Rest**: All persistent data is encrypted
- **Transport Security**: All API communication uses TLS

### API Security

- **Rate Limiting**: Prevention of brute force and DoS attacks
- **Helmet Integration**: HTTP security headers
- **Input Validation**: Thorough validation of all input parameters
- **CORS Protection**: Strict cross-origin resource sharing policies

---

## KMS Integration

The Wallet Service leverages Google Cloud KMS for cryptographic operations, providing security for cryptocurrency keys.

### Key Management

- **Key Creation**: Automated creation of asymmetric P-256 keys in KMS with SHA256 digest
- **Key Rotation**: Support for key version management
- **Key Hierarchy**: Organized key ring structure by environment
- **Access Control**: Fine-grained IAM permissions
- **Multi-Region Support**: Flexibility to use keys in different geographical regions
- **Fallback Region Support**: Service can operate with different regional key rings

### Signing Process

The transaction signing process has been extensively optimized and follows these steps:

1. **Transaction Preparation**: The transaction is serialized into the Solana message format
2. **Message Handling**: The message is base64 encoded for transmission to KMS
3. **KMS Signing**: The message is signed by the KMS service using P-256 with SHA256
4. **Signature Verification**: The signature is verified against the transaction before submission

### Error Handling

- **Retry Mechanism**: Exponential backoff for temporary KMS failures
- **Circuit Breaker**: Protection against sustained KMS outages
- **Detailed Logging**: Comprehensive error information for debugging
- **Metrics Tracking**: Monitoring of KMS operation success rates
- **Request Parameter Logging**: Detailed logging of KMS request parameters for troubleshooting
- **Enum Value Diagnostics**: Logging of KMS enum values for proper configuration verification

### Security Considerations

- **Key Accessibility**: Keys can only be used for signing, never for export
- **Audit Trail**: All cryptographic operations are logged and auditable
- **Hardware Security**: Leveraging Google's HSM backends for critical keys
- **Defense in Depth**: Multiple layers of security controls
- **Purpose-Algorithm Validation**: Ensuring correct cryptographic purpose and algorithm combinations

---

## Message Queue System

The Wallet Service uses RabbitMQ as its message broker, providing asynchronous communication with other services.

### Queue Architecture

- **Exchange Design**: Topic-based exchange for routing flexibility
- **Queue Design**: Separate request and response queues
- **Message Schema**: Structured JSON with version control
- **Correlation**: Request-response correlation via IDs

### Reliability Features

- **Connection Management**: Robust connection handling with auto-reconnection
- **Message Persistence**: Durable messages to survive broker restarts
- **Acknowledgment**: Proper message acknowledgment to prevent data loss
- **Dead Letter Handling**: Failed messages are redirected to a DLQ for analysis

### Message Processing

- **Consumer Management**: Controlled message prefetching and processing
- **Serialization**: Consistent message serialization/deserialization
- **Routing**: Intelligent message routing based on message type
- **Response Handling**: Proper correlation of responses to requests

### Performance Optimization

- **Connection Pooling**: Efficient use of connections
- **Channel Reuse**: Minimizing channel creation overhead
- **Batch Processing**: Grouping operations when possible
- **Resource Management**: Controlled resource utilization

---

## Transaction Processing

The Wallet Service implements a robust transaction processing pipeline for cryptocurrency operations.

### Transaction Creation

- **Transaction Building**: Creation of Solana transactions with proper parameters
- **Fee Calculation**: Dynamic fee estimation based on network conditions
- **Blockhash Management**: Fetching and caching of recent blockhashes
- **Input Validation**: Comprehensive validation of transaction parameters

### Signing Process

- **Secure Signing**: Remote signing via KMS, never exposing private keys
- **Signature Verification**: Verification of all signatures before submission
- **Multi-signature Support**: Handling of transactions requiring multiple signers
- **Error Handling**: Comprehensive error handling during signature process

### Transaction Submission

- **Reliable Submission**: Robust blockchain submission with retry logic
- **Confirmation Tracking**: Monitoring transaction confirmations
- **Receipt Generation**: Creating verifiable transaction receipts
- **Failure Handling**: Proper handling of submission failures

### Transaction Types

- **SOL Transfers**: Native token transfers
- **SPL Token Transfers**: Token program interactions
- **Program Interactions**: Smart contract calls
- **Custom Instructions**: Support for arbitrary instructions

---

## Monitoring & Metrics

The Wallet Service implements monitoring and metrics collection for operational visibility.

### Health Monitoring

- **Component Health Checks**: Regular verification of all system components
- **External Dependency Checks**: Monitoring of RabbitMQ, KMS, and Solana RPC
- **HTTP Health Endpoints**: `/health`, `/live`, and `/ready` endpoints for Kubernetes integration
- **Self-healing**: Automatic recovery from certain failure conditions

### Metrics Collection

- **Prometheus Integration**: Export of metrics in Prometheus format
- **Custom Metrics**: Service-specific metrics for business operations
- **System Metrics**: Node.js runtime and resource utilization metrics
- **Transaction Metrics**: Success rates, durations, and error counts

### Key Metrics

- **wallet_service_up**: Overall service health indicator
- **wallet_transaction_signing_success_rate**: Success rate of transaction signing
- **wallet_transaction_signing_duration_ms**: Duration of signing operations
- **wallet_kms_error_rate**: Rate of KMS API errors

### Alerting Considerations

- **Threshold Configuration**: Customizable alert thresholds
- **Gradual Degradation Detection**: Identifying slow-developing issues
- **Error Rate Monitoring**: Tracking of error trends over time
- **Latency Tracking**: Identifying performance degradation

---

## Error Handling & Resilience

The Wallet Service implements robust error handling and resilience patterns to maintain reliability.

### Error Handling Strategy

- **Structured Error Types**: Hierarchical error classification
- **Contextual Information**: Rich error context for debugging
- **Client-Friendly Messages**: User-appropriate error responses
- **Detailed Logging**: Comprehensive error logging
- **Operation-Specific Error Tracking**: Detailed diagnostics for KMS operations

### Resilience Patterns

- **Circuit Breaker**: Protection against cascading failures
- **Retry with Backoff**: Intelligent retry for transient failures
- **Fallback Mechanisms**: Graceful degradation when possible
- **Resource Limiting**: Prevention of resource exhaustion
- **Adaptive Retry Delays**: Dynamically calculated retry intervals based on failure type

### Failure Recovery

- **Consistent State**: Ensuring system consistency after failures
- **Graceful Shutdown**: Proper handling of termination signals
- **Connection Recovery**: Automatic reconnection to dependencies
- **Self-healing**: Automatic recovery from certain failure states

### Testing for Failures

- **Chaos Testing**: Verification of behavior under failure conditions
- **Dependency Mocking**: Testing with simulated dependency failures
- **Load Testing**: Behavior verification under high load
- **Edge Case Handling**: Testing of unusual conditions and inputs

---

## API Documentation

The Wallet Service provides a documented API for client interactions.

### Message Queue API

| Route | Description | Parameters |
|-------|-------------|------------|
| `wallet.management.create` | Create a new wallet | `userId`, `label` |
| `wallet.management.list` | List user wallets | `userId` |
| `wallet.management.get` | Get wallet details | `userId`, `walletId` |
| `wallet.management.setActive` | Set active wallet | `userId`, `walletId` |
| `wallet.balance.get` | Get wallet balance | `userId`, `walletId` |
| `wallet.balance.getTokens` | Get token balances | `userId`, `walletId` |
| `wallet.transaction.transferSol` | Transfer SOL | `userId`, `walletId`, `recipient`, `amount` |
| `wallet.transaction.sign` | Sign transaction | `userId`, `walletId`, `transaction` |

### HTTP API

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Service health check |
| `/live` | GET | Liveness probe |
| `/ready` | GET | Readiness probe |
| `/metrics` | GET | Prometheus metrics |

### Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `WALLET_NOT_FOUND` | Wallet not found or not owned by user | Verify wallet ID and ownership |
| `INSUFFICIENT_FUNDS` | Insufficient balance for transaction | Check wallet balance |
| `INVALID_RECIPIENT` | Invalid recipient address | Verify recipient address format |
| `SIGNING_FAILED` | Transaction signing failed | Check key status in KMS |
| `SUBMISSION_FAILED` | Transaction submission failed | Verify network status and retry |

---

## Testing Framework

The Wallet Service implements a testing strategy to ensure reliability.

### Testing Levels

- **Unit Tests**: Isolated testing of individual components
- **Integration Tests**: Verification of component interactions
- **System Tests**: End-to-end testing of service behavior
- **Performance Tests**: Verification of service under load

### Test Environments

- **Local Development**: For rapid development and testing
- **CI Pipeline**: Automated testing on code changes
- **Staging**: Production-like environment for integration testing
- **Production**: Canary deployments with monitoring

### Testing Tools

- **Jest**: Primary testing framework
- **Supertest**: HTTP endpoint testing
- **Mock Services**: Simulated dependencies
- **Test Data Generation**: Automated test data creation

### Test Coverage

- **Service Logic**: Core business logic
- **Failure Scenarios**: Error handling and recovery
- **Edge Cases**: Unusual inputs and conditions
- **Security**: Authentication and authorization checks

---

## Deployment Considerations

The Wallet Service is designed for container based deployment in Kubernetes.

### Configuration Management

- **Environment Variables**: Runtime configuration via environment
- **Config Maps**: Kubernetes-managed configuration
- **Secrets Management**: Secure handling of sensitive values
- **Feature Flags**: Runtime feature enablement/disablement

### Resource Requirements

- **CPU**: Moderate (1-2 cores recommended)
- **Memory**: 512MB-1GB per instance
- **Storage**: Minimal (stateless design)
- **Network**: Outbound access to KMS and Solana RPC

### Scalability

- **Horizontal Scaling**: Multiple instances for throughput
- **Load Balancing**: Even distribution of requests
- **Connection Pooling**: Efficient resource utilization
- **Rate Limiting**: Protection against traffic spikes

### Monitoring Integration

- **Prometheus**: Metrics collection
- **Liveness Probes**: Container health verification
- **Readiness Probes**: Service availability verification
- **Logging**: Structured logs for analysis

---

## Future Enhancements

The following enhancements are planned for future iterations:

### Technical Improvements

- **Multi-chain Support**: Expansion to additional blockchains
- **Enhanced Caching**: Performance optimization through caching
- **Advanced Key Rotation**: Automated key rotation policies
- **Expanded Metrics**: More detailed operational metrics

### Feature Enhancements

- **Batch Transaction Support**: Processing multiple transactions as a batch
- **Transaction History**: Maintaining a record of past transactions
- **Address Book**: Management of frequent recipient addresses
- **Advanced Token Management**: Improved token handling capabilities

### Security Enhancements

- **Advanced Threat Detection**: Anomaly detection in transaction patterns
- **Enhanced Audit Logging**: More detailed security audit trail
- **HSM Integration**: Direct hardware security module support
- **Multi-factor Authorization**: Additional security for high-value transactions

---

## Redis Integration

The Wallet Service incorporates Redis for enhanced performance and scalability.

### Caching Strategy

- **Wallet Information Caching**: Frequently accessed wallet data is cached to reduce storage I/O
- **Blockchain Data Caching**: Network-retrieved data like account info and token balances are cached
- **Rate Limit Enforcement**: Redis-based rate limiting to prevent API abuse
- **Distributed Locking**: Coordination of concurrent operations across service instances
- **Circuit Breaker State**: Shared circuit breaker state across service instances

### Performance Optimizations

- **Connection Pooling**: Efficient reuse of Redis connections
- **Pipelining**: Batched Redis operations for reduced network overhead
- **Serialization Strategy**: Optimized data serialization for minimal memory usage
- **TTL Management**: Appropriate time-to-live settings for different data types
- **Memory Usage Monitoring**: Tracking Redis memory consumption

### Resilience Features

- **Reconnection Logic**: Automatic reconnection after Redis connectivity issues
- **Command Timeouts**: Prevention of blocking operations
- **Replica Failover**: Smooth transition during Redis primary node failures
- **Cache Stampede Prevention**: Strategies to prevent concurrent cache rebuilds
- **Degraded Operation**: Ability to function with reduced capabilities when Redis is unavailable

### Security Considerations

- **TLS Encryption**: Secure communication with Redis
- **Authentication**: Password-based Redis authentication
- **Network Isolation**: Redis deployed in private network
- **Data Sanitization**: Careful validation of all cached data
- **No Sensitive Data**: Ensuring sensitive information is never cached

---

*This document will be updated as the project evolves. Last updated: March 10, 2025.* 
