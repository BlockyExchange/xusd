# XUSD Stack Docker Compose Setup

## Overview
XUSD is the digital stablecoin pegged to the Brazilian Real, built on XNO (Nano) technology, providing asynchronous, instantaneous, and feeless transactions, similar to Brazil's PIX payment system.

This repository contains a ready-to-use `docker-compose.yaml` file to run the full XUSD ecosystem, provided by Blocky Exchange (BLOCKY SERVICOS DIGITAIS LTDA). You can either use prebuilt images or build them yourself from the source.

## Quick Start

To start quickly using prebuilt images from Blocky's public registry:

```bash
git clone https://github.com/BlockyExchange/xusd
cd xusd
docker compose up -d
# or
podman compose up -d
```

### Building from Source
If you prefer building the images from source:

```bash
git clone https://github.com/BlockyExchange/xusd --recursive
cd xusd
docker compose build
# or
podman compose build
```

## Services

| Service                 | Description                                                                                 | Ports                                | Exposed                   |
|-------------------------|---------------------------------------------------------------------------------------------|--------------------------------------|---------------------------|
| `xusd-node`             | The main XUSD node.                                                                         | 7025 (peering), 7026 (RPC), 7028 (WebSocket) | 7025 publicly; RPC/WebSocket behind proxy |
| `xusd-explorer`         | Blockchain explorer for XUSD                                                                | 80                                   | Behind xusd-proxy         |
| `xusd-wallet`           | Web wallet interface for XUSD                                                               | 80                                   | Behind xusd-proxy         |
| `xusd-work-server`      | PoW work server (AMD ROCm supported)                                                        | 7023                                 | Publicly via RPC proxy    |
| `xusd-pippin`           | High-performance internal wallet (for apps)                                                 | 7029                                 | Internal only             |
| `xusd-blockycon`        | Service serving dynamic avatars for wallets                                                 | 5000                                 | Behind xusd-proxy         |
| `xusd-node-rpc-proxy`   | RPC/WebSocket proxy for external node interactions                                          | 9950 (RPC), 9952 (WebSocket)         | Publicly exposed          |
| `xusd-proxy`            | Reverse proxy (Caddy) routing to wallet, explorer, blockycon                                | 80                                   | Exposed for reverse proxy |
| Redis/MongoDB Services  | Database/cache dependencies                                                                 | Default                              | Internal only             |

## GPU Support

The `xusd-work-server` container is configured by default for AMD GPUs using ROCm (compatible with AMD Radeon 5500 XT). If using an NVIDIA GPU or other systems, you must create a custom Docker image from [nano-work-server](https://github.com/nanocurrency/nano-work-server).

For AMD GPUs, follow the [official ROCm setup guide](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/prerequisites.html).

## Becoming a Representative

If you are a Blocky partner, a bank, an institution, or a company building on top of the XUSD network, you may be interested in participating as a Principal Representative and helping to decentralize the XUSD network.

To participate in the Open Representative Voting Process, your node can be assigned voting weight, allowing you to take part in important network decisions. This process helps to ensure a democratic and decentralized governance model for XUSD.

### How to Become a Representative

If you're interested in becoming a Principal Representative for XUSD:
- Contact us: Get in touch with the Blocky team for further information.

- Set up a Voting Node: Your node will need to have the enable_voting option set to true in the config-node.toml file.
  
- Delegate Voting Weight: Once your node is ready, we will assign voting weight to your node, allowing it to participate in the network's decision-making process.

For more details and to get started, visit blocky.com.br or reach out to our support team.

## Configuration Files

### Node Configuration (`./config/xusd-node`)

- `config-rpc.toml`:
  ```toml
  address = "::ffff:0.0.0.0"
  enable_control = false # Set true cautiously, risk of funds/control loss
  ```

- `config-node.toml`:
  ```toml
  [node.websocket]
  address = "::ffff:0.0.0.0"
  enable = true

  [rpc]
  enable = true

  [node]
  enable_voting = true # Set false for lightweight nodes, true for representatives
  ```

Copy these configurations into `./data/xusd-node-data`.

### RPC Proxy Configuration (`./config/xusd-node-rpc-proxy`)
- `settings.json`: Manages RPC/WS connections, rate limits, and DOS protection.
- `pow_creds.json`: Defines the PoW server endpoints.

### Proxy Configuration (`./config/xusd-proxy`)
`Caddyfile` manages reverse proxy settings:

```caddyfile
http://wallet.xusd.blocky.com.br {
    encode gzip zstd
    reverse_proxy http://xusd-wallet:80
}

http://explorer.xusd.blocky.com.br {
    encode gzip zstd
    reverse_proxy http://xusd-explorer:80
}

http://blockycon.blocky.com.br {
    encode gzip zstd
    reverse_proxy http://xusd-blockycon:5000
}
```

Adjust for TLS as needed.

## Flexible Usage
You can selectively run services based on your needs:

- **Full Stack:** Node, explorer, wallet, PoW server, Pippin, and proxies.
- **Minimal setup:** Node + Pippin for programmatic interaction.
- **Explorer-only:** Node, explorer, and dependencies.

## Health Checks
Each service includes health checks ensuring robust operation, including a custom Perl-based health check for `xusd-node`.

## Support and Contact
If you encounter any issues or have further questions, please contact Blocky support.

## Use XUSD

Visit [blocky.com.br](https://blocky.com.br) to create your account, deposit Brazilian Reais via PIX, and receive XUSD stablecoins instantly. You can withdraw XUSD to your wallet or deposit XUSD back into your account for seamless crypto trading.

