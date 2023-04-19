# Omnia

[![Omnia Tests](https://github.com/chronicleprotocol/omnia/actions/workflows/test.yml/badge.svg)](https://github.com/chronicleprotocol/omnia/actions/workflows/test.yml)
[![Build & Push Docker Image](https://github.com/chronicleprotocol/omnia/actions/workflows/docker.yml/badge.svg)](https://github.com/chronicleprotocol/omnia/actions/workflows/docker.yml)

For more information on running oracles see: https://github.com/chronicleprotocol/oracles

## Working with Docker

We introduced Docker environment for local omnia development.

Please follow several steps to build and run it.

#### Building omnia image

NOTE: You have to build it from repo root.

`docker build -t omnia .` OR `docker-compose build`

To run `omnia` in container from prev command you can use `omnia` command:

```bash
$ omnia
Importing configuration from /home/omnia/config/feed.json...
```

## Docker compose
For even more easy development we providing you with `docker-compose.yml` file that will help to set everything up for you.

Right now it contains `omnia_feed` container. 
It contains working feed configuration + spire integration.

And `spire` container with configured spire agent that will be called from `omnia_feed`.

Example of usage: 

1. Starting Spire Agent

```bash
$ docker-compose up -d spire omnia gofer
```

## Docker integration

#### Building Omnia with custom dependencies

For production use we building Omnia with predefined locked to tags dependencies:

| Dependency  | Version               | Argument name      | Repository                                        |
|-------------|-----------------------|--------------------|---------------------------------------------------|
| Eth Sign    | `tags/ethsign/0.17.0` | `ETHSIGN_REF`      | https://github.com/dapphub/dapptools              |
| Gofer       | `tags/v0.4.6`         | `ORACLE_SUITE_REF` | https://github.com/chronicleprotocol/oracle-suite |
| Spire       | `tags/v0.4.6`         | `ORACLE_SUITE_REF` | https://github.com/chronicleprotocol/oracle-suite |
| Setzer      | `tags/v0.4.2`         | `SETZER_REF`       | https://github.com/chronicleprotocol/setzer       |

We using git reference format to be able to use custom versions/commits for DEV builds.
If you need to build omnia with custom version you can use [Docker ARGs](https://docs.docker.com/engine/reference/builder/#arg) to do this.

Example: 


#### Omnia Configuration

Dockerized Omnia default configuration:

| Env Var        | Default value            | Description                        |
|----------------|--------------------------|------------------------------------|
| `OMNIA_CONFIG` | `/home/omnia/omnia.json` | Omnia configuration file           |
| `SPIRE_CONFIG` | `/home/omnia/spire.json` | Spire configuration file           |
| `GOFER_CONFIG` | `/home/omnia/gofer.json` | Gofer configuration file           |
| `ETH_RPC_URL`  | `http://geth.local:8545` | Setzer requirement for wstETH pair |
| `ETH_GAS`      | `7000000`                | Gofer configuration file           |


To set custom configuration values use [ENV (environment variables)](https://docs.docker.com/engine/reference/run/#env-environment-variables)

Configuration files might be provided by mounting it into Docker container. 




### Configuring Spire

>spire is installed through docker.

This is based on libp2p which is a peer-to-peer networking protocol designed to enable decentralized communication and file sharing over the internet. It is a modular, open-source networking protocol that allows nodes to communicate with each other directly, without the need for a central server or infrastructure.


 One of these attributes is the "transport" attribute, which uses libp2p to establish direct peer-to-peer connections between nodes without relying on a central server or infrastructure, So we have to give the listenAddrs for example `/ip4/192.168.18.109/tcp/37705/p2p/12D3KooWPFpaE13gph8p6jdNGJv1M6fwDro8kdst53MUzVpuSJUL` i.e **"\<ip-version>/\<host>/\<protocol>/\<port>/\<type>/\<peer_id>"** w.r.t the quorum of median.To obtain the peer addresses, you can check the logs of Spire using journalctl, as Spire runs as a systemd service. Once you have the peer addresses, then ** you can add them to the "directPeersAddrs" array to connect peers in the "transport" attribute of the Spire configuration file. **

```
sudo journalctl -u <spire-agent.service> -n 100 -b -f
```


**add peerIDs in this attr to connect spire with each other**
```json 
"transport":{
      "libp2p": {
        "directPeersAddrs":[]}}
  ```

### command to run spire
```
spire agent -c <CONFIG_PATH> --log.verbosity debug
```
### command to run spire systemd
```
systemctl start <spire-agent.service>

```

### command to run ssb-server systemd
```
systemctl start <ssb-server.service>

```

The installed Scuttlebot config can be found in `~/.ssb.config`, more details
about the [Scuttlebot config](https://github.com/ssbc/ssb-config#configuration).

### Creating and Accepting SSB Invites 

Open the systemd file of ssb-server.service to lookup the binary of ssb-server.

Creating an SSB Invite

`ssb-server invite.create 1` means `<path of ssb-server> invite.create 1`

This will output a JSON object containing the invite code.

Accepting an SSB Invite

To accept an SSB invite without using Docker, open a terminal window and run the following command:

`ssb-server invite.accept <invite_code>`

Replace <invite_code> with the invite code obtained in the previous step.


### how it will work

 we should run the 3 feeds with the 3 spires 
 every feed should have their own omnia
 means the config file have the right omnia addr pasted in feed object of spire's config



