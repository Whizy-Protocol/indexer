# Whizy Protocol - Indexer

A blockchain event indexer for the Whizy Protocol, designed to index smart contract events from the HEDERA blockchain network into a PostgreSQL database. The indexer monitors prediction market events and protocol-related activities.

## Overview

The Whizy Protocol Indexer is a Go-based application that continuously monitors blockchain events from smart contracts and stores them in a database for efficient querying. It supports multiple contract types and provides reliable event synchronization with automatic recovery capabilities.

## Features

- **Multi-contract indexing**: Supports indexing from multiple smart contracts simultaneously
- **Event parsing**: Parses and stores various event types from prediction markets and protocol operations
- **Database synchronization**: Maintains sync state for each contract to ensure data consistency
- **Batch processing**: Processes blocks in configurable batches for optimal performance
- **Graceful shutdown**: Handles shutdown signals and saves processing state
- **Auto-migration**: Automatically creates and updates database tables
- **Configurable workers**: Supports multiple indexing workers for parallel processing

## Supported Events

The indexer tracks the following blockchain events:

### Prediction Market Events
- **BetPlaced**: Records when users place bets on prediction markets
- **MarketCreated**: Tracks creation of new prediction markets
- **MarketResolved**: Records market resolution outcomes
- **WinningsClaimed**: Tracks when users claim their winnings

### Protocol Events
- **AutoDepositExecuted**: Records automatic deposit operations
- **AutoWithdrawExecuted**: Records automatic withdrawal operations
- **ProtocolRegistered**: Tracks protocol registrations
- **ProtocolUpdated**: Records protocol parameter updates
- **OwnershipTransferred**: Tracks ownership changes
- **Paused/Unpaused**: Records contract pause state changes

## Requirements

- Go 1.24 or higher
- PostgreSQL database
- Access to HEDERA blockchain RPC endpoint

## Installation

### From Source

```bash
git clone <repository-url>
cd indexer
go mod download
go build -o go-indexer
```

### Using Docker

```bash
docker build -t whizy-indexer .
```

## Configuration

The indexer uses YAML configuration files. Copy `config-example.yaml` to `config.yaml` and modify as needed

### Network Configuration

Define contract addresses and start blocks in `networks.json`:

```json
{
  "hedera-testnet": {
    "WhizyPredictionMarket": {
      "address": "0x71711F35c200fDabE75F2e82F0146c35f32eBAA5",
      "startBlock": 60892524
    },
    "ProtocolSelector": {
      "address": "0x5F9fb4Ac021Fc6dD4FFDB3257545651ac132651C",
      "startBlock": 60892524
    }
  }
}
```

## Database Setup

### PostgreSQL Setup

1. Create a PostgreSQL database:
```sql
CREATE DATABASE "whizy-indexer-base";
```

2. Ensure your user has appropriate permissions:
```sql
GRANT ALL PRIVILEGES ON DATABASE "whizy-indexer-base" TO your_username;
```

### Schema

The indexer automatically creates the following tables:
- `bet_placeds`
- `market_createds`
- `market_resolveds`
- `winnings_claimeds`
- `auto_deposit_executeds`
- `auto_withdraw_executeds`
- `ownership_transferreds`
- `pauseds`
- `protocol_registereds`
- `protocol_updateds`
- `unpauseds`
- `sync_states`

## Usage

### Running the Indexer

```bash
# Run with default config
./go-indexer

# Run with custom config file
./go-indexer -config custom-config.yaml
```

### Docker Usage

```bash
docker run -v $(pwd)/config.yaml:/app/config.yaml \
           -v $(pwd)/networks.json:/app/networks.json \
           whizy-indexer
```

### Monitoring

The indexer provides console output for monitoring:
- Contract loading status
- Block processing progress
- Event parsing and storage statistics
- Error reporting and recovery attempts

## Architecture

### Components

- **Main Process**: Handles configuration loading, database initialization, and process coordination
- **Indexer**: Core indexing logic that processes blockchain events
- **RPC Client**: Manages blockchain RPC connections and queries
- **Parser**: Decodes and transforms blockchain events into database entities
- **Config Manager**: Handles configuration and network definitions
- **Database Layer**: GORM-based database operations with automatic migrations

### Processing Flow

1. Load configuration and network definitions
2. Initialize database connection and perform migrations
3. Start indexing workers for each configured contract
4. Continuously fetch and process blocks in batches
5. Parse events and store in database with conflict resolution
6. Update sync state to track progress
7. Handle graceful shutdown with state preservation

## Error Handling

- **Connection Recovery**: Automatically retries failed RPC connections
- **Block Reprocessing**: Retries failed block processing with exponential backoff
- **Data Integrity**: Uses database constraints and conflict resolution
- **State Preservation**: Saves processing state on shutdown for recovery

## Performance Considerations

- **Batch Processing**: Configurable batch sizes for optimal throughput
- **Parallel Workers**: Multiple workers process different contracts simultaneously
- **Database Indexing**: Appropriate indexes on frequently queried columns
- **Memory Management**: Efficient event processing with minimal memory footprint

## Development

### Project Structure

```
.
├── main.go                 # Application entry point
├── config/                 # Configuration management
│   ├── config.go          # Configuration structures and loading
│   └── db.go              # Database models and connection
├── indexer/               # Core indexing logic
│   ├── indexer.go         # Main indexing orchestration
│   ├── rpc.go             # RPC client implementation
│   └── parser.go          # Event parsing logic
├── config.yaml            # Main configuration file
├── networks.json          # Network and contract definitions
└── Dockerfile             # Container build configuration
```

### Building

```bash
# Build for current platform
go build -o go-indexer

# Build for Linux
GOOS=linux GOARCH=amd64 go build -o go-indexer-linux

# Build with Docker
docker build -t whizy-indexer .
```

## Troubleshooting

### Common Issues

1. **Database Connection Errors**
   - Verify PostgreSQL is running and accessible
   - Check database credentials and permissions
   - Ensure database exists

2. **RPC Connection Issues**
   - Verify RPC endpoint is accessible
   - Check network connectivity
   - Validate RPC endpoint supports required methods

3. **Sync Issues**
   - Use `forceResyncOnEveryStart: true` to restart from beginning
   - Check `sync_states` table for current progress
   - Verify start block numbers in network configuration

### Logging

The indexer provides detailed logging including:
- Configuration loading status
- Database connection and migration status
- Contract processing progress
- Event parsing and storage statistics
- Error messages with context

## Contributing

Contributions are welcome. Please ensure:
- Code follows Go best practices
- Database migrations are backward compatible
- Tests are included for new functionality
- Documentation is updated for new features

## License

[License information to be added]
