# ansible-role-lotus

Ansible role that install and configures Filecoin's Lotus daemon and miner.

Before using this Ansible role please familiarize yourself with the Filecoin Project by reading the [Official Filecoin Docs](https://docs.filecoin.io/)

This role will install Filecoin's Lotus Daemon and (optionally) miner from the official source code. By default only the Lotus daemon is enabled.

## Supported platforms

- Ubuntu/Debian

## Requirements

- **Required ansible version:** >= 2.9

For `lotus` requirements please [check the official documentation](https://docs.filecoin.io/get-started/lotus/installation/#minimal-requirements).

## Installation

### Basic installation

To proceed with the default installation which only enables the Lotus daemon, you can use the following playbook:

```yaml
---
- hosts: localhost
  roles:
     - role: DigitalMOB.lotus
       lotus_git_version: v1.2.1
```       

### Custom build environment

At build time this role sets `RUSTFLAGS` and `FFI_BUILD_FROM_SOURCE`. You can add change these environment variables, remove them or add more if needed:

```yaml
---
- hosts: localhost
  roles:
     - role: DigitalMOB.lotus
       lotus_git_version: v1.2.1
       lotus_build_env:
         RUSTFLAGS: "-C target-cpu=native"
         FFI_BUILD_FROM_SOURCE: 1
```       

### Mining

#### Enabling the miner

If you wish to enable mining, you can do so by enabling the miner and providing actor, owner and miner addresses:

```yaml
---
- hosts: localhost
  roles:
     - role: DigitalMOB.lotus
       lotus_git_version: v1.2.1
       lotus_miner_enabled: yes
```       

Once the miner starts you will have to initialize the miner by following steps in the [official Filecoin documentation](https://docs.filecoin.io/mine/lotus/miner-setup/#miner-initialization)

#### Initializing the miner

If you already know the actor adress and owner and worker keys, the miner can be initialized by the role itself automatically:

```yaml
---
- hosts: localhost
  roles:
     - role: DigitalMOB.lotus
       lotus_git_version: v1.2.1
       lotus_miner_enabled: yes
       lotus_miner_actor_address: REDACTED
       lotus_miner_owner_address: REDACTED
       lotus_miner_worker_address: REDACTED
```       

When you chose to do so, the lotus miner service will not be started automatically. Once the miner is initialized you will have to start the service manually:

```sh
systemctl start lotus-miner
```

### Configuration files

Configuration (`config.toml`) files for both the daemon and the miner can be overriden if needed:

```yaml
---
- hosts: localhost
  roles:
     - role: DigitalMOB.lotus
       lotus_git_version: v1.2.1
       lotus_miner_enabled: yes
       lotus_daemon_config: |
         [API]
           ListenAddress: "/ip4/0.0.0.0/tcp/1234/http"
       lotus_miner_config: |
         [API]
           ListenAddress = "/ip4/0.0.0.0/tcp/2345/http"
         [Libp2p]
           ListenAddresses = ["/ip4/0.0.0.0/tcp/3452", "/ip6/::/tcp/0"]
         [Dealmaking]
           ConsiderOnlineStorageDeals = true
           ConsiderOfflineStorageDeals = true
           ConsiderOnlineRetrievalDeals = true
           ConsiderOfflineRetrievalDeals = true
```       

### Runtime Environment Variables

Runtime environment variables can be added if needed:

```yaml
---
- hosts: localhost
  roles:
     - role: DigitalMOB.lotus
       lotus_git_version: v1.2.1
       lotus_miner_enabled: yes
       lotus_daemon_env:
         DAEMON_VAR: value
       lotus_miner_env:
         MINER_VAR: value
```

## Role Variables

| Parameter    | Default |  Description |
|--------------|----------|-------------|
|`lotus_git_repo`|`https://github.com/filecoin-project/lotus.git`| Lotus Git repository |
|`lotus_git_version`|`v1.2.1`| Git tag, branch or hash to use for building lotus. |
|`lotus_user`|`lotus`| System username which will run the lotus daemon and miner. |
|`lotus_user_uid`|`532`| Lotus username UID. |
|`lotus_golog_log_fmt`|`json`| Lotus logs formating. Applies to both daemon and miner/ |
|`lotus_proof_params_path`|`/var/tmp/filecoin-proof-params`| Path where to store Filecoin Proof Parameters. |
|`lotus_ipfs_gateway`|`https://proofs.filecoin.io/ipfs`| IPFS gateway to use for downloading Proof Parameters. |
|`lotus_build_env`|`{'RUSTFLAGS': '-C target-cpu=native -g', 'FFI_BUILD_FROM_SOURCE': 1}`| Compiler flags to use for building lotus. |
|`lotus_daemon_enabled`|`yes`| Whether or not to build and run the Lotus daemon. |
|`lotus_daemon_path`|`/var/lib/lotus`| Lotus daemon data store path. |
|`lotus_daemon_golog_file`|`/var/log/lotus/daemon.log`| Lotus daemon log file location. |
|`lotus_daemon_env`|`{}`| Extra environment variables to be passed to Lotus daemon at startup. See [the official documentation](https://docs.filecoin.io/get-started/lotus/configuration-and-advanced-usage/#environment-variables) for supported values. |
|`lotus_daemon_systemd_extras`|`{}`| Extra systemd parameters to be passed to the systemd service file. |
|`lotus_daemon_config`|`[API]\n  ListenAddress: "/ip4/0.0.0.0/tcp/1234/http"`| Lotus daemon `config.toml` file. See [the official documentation](https://docs.filecoin.io/get-started/lotus/configuration-and-advanced-usage/#configuration) for reference. |
|`lotus_miner_enabled`|`no`| Wheter or not to build and run the Lotus miner. |
|`lotus_miner_init`|`no`| Set to yes if you want to initialize your miner during the bootstrap process. This requires a valid `lotus_miner_addr` and `lotus_miner_wallet_address`. |
|`lotus_miner_path`|`/var/lib/lotusminer`| Lotus miner data store path. |
|`lotus_miner_golog_file`|`/var/log/lotus/miner.log`| Lotus miner log file location. |
|`lotus_miner_env`|`{}`| Extra environment variables to be passed to Lotus miner at startup. See [the official documentation](https://docs.filecoin.io/get-started/lotus/configuration-and-advanced-usage/#environment-variables) for supported values.|
|`lotus_miner_systemd_extras`|`{}`| Extra systemd parameters to be passed to the systemd service file. |
|`lotus_miner_config`|`[API]\n  ListenAddress = "/ip4/0.0.0.0/tcp/2345/http"\n[Libp2p]\n  ListenAddresses = ["/ip4/0.0.0.0/tcp/3452", "/ip6/::/tcp/0"]\n [Dealmaking]\n  ConsiderOnlineStorageDeals = true\n  ConsiderOfflineStorageDeals = true\n ConsiderOnlineRetrievalDeals = true\n ConsiderOfflineRetrievalDeals = true`| Lotus miner `config.toml` file. See [the official documentation](https://docs.filecoin.io/get-started/lotus/configuration-and-advanced-usage/#configuration) for reference.| 
|`lotus_miner_actor_address`|`''` | Lotus miner actor address. Required if `lotus_miner_init` is set to `yes`. |
|`lotus_miner_owner_address`|`''`| Lotus miner owner address. Required if `lotus_miner_init` is set to `yes`. |
|`lotus_miner_worker_address`|`''`| Lotus miner worker address. Required if `lotus_miner_init` is set to `yes`. |
|`destroy_lotus_daemon`|`no`| !!! DANGER: Completely removes **ALL** Lotus daemon data !!! |
|`destroy_lotus_miner` | `no` | !!! DANGER: Completely removes **ALL** Lotus miner data !!! |

## License

MIT

## Author Information

This role was created by liviudm.
