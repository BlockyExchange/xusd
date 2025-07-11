# XBRL Stack Docker Compose Setup

## Overview
XBRL is the digital stablecoin pegged to the Brazilian Real, built on XNO (Nano) technology, providing asynchronous, instantaneous, and feeless transactions, similar to Brazil's PIX payment system.

This repository contains a ready-to-use `docker-compose.yaml` file to run the full XBRL ecosystem, provided by Blocky Exchange (BLOCKY SERVICOS DIGITAIS LTDA). You can either use prebuilt images or build them yourself from the source.

## Quick Start

To start quickly using prebuilt images from Blocky's public registry:

```bash
git clone https://github.com/BlockyExchange/xbrl
cd xbrl
docker compose up -d
# or
podman compose up -d
```

### Building from Source
If you prefer building the images from source:

```bash
git clone https://github.com/BlockyExchange/xbrl --recursive
cd xbrl
docker compose build
# or
podman compose build
```

## Services

| Service                 | Description                                                                                 | Ports                                | Exposed                   |
|-------------------------|---------------------------------------------------------------------------------------------|--------------------------------------|---------------------------|
| `xbrl-node`             | The main XBRL node.                                                                         | 7015 (peering), 7016 (RPC), 7018 (WebSocket) | 7015 publicly; RPC/WebSocket behind proxy |
| `xbrl-explorer`         | Blockchain explorer for XBRL                                                                | 80                                   | Behind xbrl-proxy         |
| `xbrl-wallet`           | Web wallet interface for XBRL                                                               | 80                                   | Behind xbrl-proxy         |
| `xbrl-work-server`      | PoW work server (AMD ROCm supported)                                                        | 7013                                 | Publicly via RPC proxy    |
| `xbrl-pippin`           | High-performance internal wallet (for apps)                                                 | 7019                                 | Internal only             |
| `xbrl-blockycon`        | Service serving dynamic avatars for wallets                                                 | 5000                                 | Behind xbrl-proxy         |
| `xbrl-node-rpc-proxy`   | RPC/WebSocket proxy for external node interactions                                          | 9950 (RPC), 9952 (WebSocket)         | Publicly exposed          |
| `xbrl-proxy`            | Reverse proxy (Caddy) routing to wallet, explorer, blockycon                                | 80                                   | Exposed for reverse proxy |
| Redis/MongoDB Services  | Database/cache dependencies                                                                 | Default                              | Internal only             |

## GPU Support

The `xbrl-work-server` container is configured by default for AMD GPUs using ROCm (compatible with AMD Radeon 5500 XT). If using an NVIDIA GPU or other systems, you must create a custom Docker image from [nano-work-server](https://github.com/nanocurrency/nano-work-server).

For AMD GPUs, follow the [official ROCm setup guide](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/prerequisites.html).

## Becoming a Representative

If you are a Blocky partner, a bank, an institution, or a company building on top of the XBRL network, you may be interested in participating as a Principal Representative and helping to decentralize the XBRL network.

To participate in the Open Representative Voting Process, your node can be assigned voting weight, allowing you to take part in important network decisions. This process helps to ensure a democratic and decentralized governance model for XBRL.

### How to Become a Representative

If you're interested in becoming a Principal Representative for XBRL:
- Contact us: Get in touch with the Blocky team for further information.

- Set up a Voting Node: Your node will need to have the enable_voting option set to true in the config-node.toml file.
  
- Delegate Voting Weight: Once your node is ready, we will assign voting weight to your node, allowing it to participate in the network's decision-making process.

For more details and to get started, visit blocky.com.br or reach out to our support team.

## Configuration Files

### Node Configuration (`./config/xbrl-node`)

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

Copy these configurations into `./data/xbrl-node-data`.

### RPC Proxy Configuration (`./config/xbrl-node-rpc-proxy`)
- `settings.json`: Manages RPC/WS connections, rate limits, and DOS protection.
- `pow_creds.json`: Defines the PoW server endpoints.

### Proxy Configuration (`./config/xbrl-proxy`)
`Caddyfile` manages reverse proxy settings:

```caddyfile
http://wallet.xbrl.blocky.com.br {
    encode gzip zstd
    reverse_proxy http://xbrl-wallet:80
}

http://explorer.xbrl.blocky.com.br {
    encode gzip zstd
    reverse_proxy http://xbrl-explorer:80
}

http://blockycon.blocky.com.br {
    encode gzip zstd
    reverse_proxy http://xbrl-blockycon:5000
}
```

Adjust for TLS as needed.

## Flexible Usage
You can selectively run services based on your needs:

- **Full Stack:** Node, explorer, wallet, PoW server, Pippin, and proxies.
- **Minimal setup:** Node + Pippin for programmatic interaction.
- **Explorer-only:** Node, explorer, and dependencies.

## Health Checks
Each service includes health checks ensuring robust operation, including a custom Perl-based health check for `xbrl-node`.

## Support and Contact
If you encounter any issues or have further questions, please contact Blocky support.

## Use XBRL

Visit [blocky.com.br](https://blocky.com.br) to create your account, deposit Brazilian Reais via PIX, and receive XBRL stablecoins instantly. You can withdraw XBRL to your wallet or deposit XBRL back into your account for seamless crypto trading.

