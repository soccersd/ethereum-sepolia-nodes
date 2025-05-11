# Ethereum Node Setup Guide

This guide explains how to set up Ethereum nodes (Geth and Prysm) on a VPS.

## Prerequisites

- A VPS with at least 8GB RAM and 500GB SSD
- Docker and Docker Compose installed
- Basic terminal knowledge

## Setup Instructions

### 1. Create Required Directories

```bash
# Create directories for data storage
mkdir data jwt data-prysm
```

### 2. Create JWT Secret

```bash
# Generate a JWT secret for Geth
openssl rand -hex 32 > jwt/jwt.hex
```

### 3. Configure Docker Compose Files

Create two separate Docker Compose files:

**File: docker-compose-geth.yml**
```yaml
version: '3.8'
services:
  geth:
    image: ethereum/client-go:stable
    command: >
      --sepolia
      --http
      --http.addr 0.0.0.0
      --http.port 8545
      --http.api eth,net,web3,engine
      --ws
      --ws.addr 0.0.0.0
      --ws.port 8546
      --authrpc.vhosts=localhost
      --authrpc.jwtsecret=/jwt/jwt.hex
      --authrpc.addr 0.0.0.0
      --authrpc.port 8551
      --metrics
      --metrics.addr 0.0.0.0
      --metrics.port 6060
      --datadir /data
    volumes:
      - ./data:/data
      - ./jwt:/jwt
    ports:
      - "8545:8545"
      - "8546:8546"
      - "8551:8551"
      - "30303:30303"
      - "30303:30303/udp"
      - "6060:6060"
    restart: unless-stopped
```

**File: docker-compose-prysm.yml**
```yaml
version: '3.8'
services:
  beacon:
    image: gcr.io/prysmlabs/prysm-beacon:stable
    command: >
      --sepolia
      --datadir=/data
      --checkpoint-sync-url=https://sepolia.checkpoint-sync.ethdevops.io
      --genesis-beacon-api-url=https://sepolia.checkpoint-sync.ethdevops.io
      --accept-terms-of-use
      --rpc-host=0.0.0.0
      --rpc-port=4000
      --grpc-gateway-host=0.0.0.0
      --grpc-gateway-port=3500
      --p2p-tcp-port=13000
      --p2p-udp-port=12000
      --monitoring-host=0.0.0.0
      --monitoring-port=8080
      --fallback-web3provider=https://sepolia.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161
    volumes:
      - ./data-prysm:/data
    ports:
      - "4000:4000"
      - "3500:3500"
      - "13000:13000"
      - "12000:12000/udp"
      - "8080:8080"
    restart: unless-stopped
```

### 4. Start the Nodes

Start Geth:
```bash
docker-compose -f docker-compose-geth.yml up -d
```

Start Prysm:
```bash
docker-compose -f docker-compose-prysm.yml up -d
```

### 5. Check Node Status

Check Geth logs:
```bash
docker-compose -f docker-compose-geth.yml logs -f
```

Check Prysm logs:
```bash
docker-compose -f docker-compose-prysm.yml logs -f
```

### 6. Stop the Nodes

Stop Geth:
```bash
docker-compose -f docker-compose-geth.yml down
```

Stop Prysm:
```bash
docker-compose -f docker-compose-prysm.yml down
```

## Important Notes

- Geth will sync the Sepolia testnet blockchain (this may take several hours)
- Prysm uses checkpoint syncing to speed up the initial sync
- Both nodes are independent and don't communicate with each other
- Geth data is stored in the `./data` directory
- Prysm data is stored in the `./data-prysm` directory
