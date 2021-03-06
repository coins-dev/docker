## Usage

### How to use this image

This image contains the main binaries from the Sugarchain Yumekawa project - `sugarchaind`, `sugarchain-cli` and `sugarchain-tx`. It behaves like a binary, so you can pass any arguments to the image and they will be forwarded to the`sugarchaind` binary:

```bash
❯ docker run --rm -it sugarchain/sugarchain \
  -printtoconsole \
  -regtest=1 \
  -rpcallowip=172.17.0.0/16
```

By default, `sugarchaind` will run as user `sugarchain` for security reasons and with its default data dir (`~/.sugarchain`). If you'd like to customize where `sugarchaind` stores its data, you must use the `SUGAR_DATA` environment variable. The directory will be automatically created with the correct permissions for the `sugarchain` user and `sugarchaind` automatically configured to use it.

```bash
❯ docker run --env SUGAR_DATA=/var/lib/sugarchain --rm -it sugarchain/sugarchain \
  -printtoconsole \
  -regtest=1
```

You can also mount a directory in a volume under `/home/sugarchain/.sugarchain` in case you want to access it on the host:

```bash
❯ docker run -v ${PWD}/data:/home/sugarchain/.sugarchain -it --rm sugarchain/sugarchain \
  -printtoconsole \
  -regtest=1
```

You can optionally create a service using `docker-compose`:

```yml
sugarchain:
  image: sugarchain/sugarchain
  command:
    -printtoconsole
    -regtest=1
```

### Using RPC to interact with the daemon

There are two communications methods to interact with a running Sugarchain Yumekawa daemon.

The first one is using a cookie-based local authentication. It doesn't require any special authentication information as running a process locally under the same user that was used to launch the Sugarchain Yumekawa daemon allows it to read the cookie file previously generated by the daemon for clients. The downside of this method is that it requires local machine access.

The second option is making a remote procedure call using a username and password combination. This has the advantage of not requiring local machine access, but in order to keep your credentials safe you should use the newer `rpcauth` authentication mechanism.

#### Using cookie-based local authentication

Start by launch the Sugarchain Yumekawa daemon:

```bash
❯ docker run --rm --name sugarchain-server -it sugarchain/sugarchain \
  -printtoconsole \
  -regtest=1
```

Then, inside the running `sugarchain-server` container, locally execute the query to the daemon using `sugarchain-cli`:

```bash
❯ docker exec --user sugarchain sugarchain-server sugarchain-cli -regtest getmininginfo

{
  "blocks": 0,
  "currentblocksize": 0,
  "currentblockweight": 0,
  "currentblocktx": 0,
  "difficulty": 4.656542373906925e-10,
  "errors": "",
  "networkhashps": 0,
  "pooledtx": 0,
  "chain": "regtest"
}
```

In the background, `sugarchain-cli` read the information automatically from `/home/sugarchain/.sugarchain/regtest/.cookie`. In production, the path would not contain the regtest part.

### Exposing Ports

Depending on the network (mode) the Sugarchain Yumekawa daemon is running as well as the chosen runtime flags, several default ports may be available for mapping.

Ports can be exposed by mapping all of the available ones (using `-P` and based on what `EXPOSE` documents) or individually by adding `-p`. This mode allows assigning a dynamic port on the host (`-p <port>`) or assigning a fixed port `-p <hostPort>:<containerPort>`.

Example for running a node in `regtest` mode mapping JSON-RPC/REST (45339) and P2P (45340) ports:

```bash
docker run --rm -it \
  -p 45339:45339 \
  -p 45340:45340 \
  sugarchain/sugarchain \
  -printtoconsole \
  -regtest=1 \
  -rpcallowip=172.17.0.0/16 \
  -rpcbind=0.0.0.0 \
  -rpcauth='foo:7d9ba5ae63c3d4dc30583ff4fe65a67e$9e3634e81c11659e3de036d0bf88f89cd169c1039e6e09607562d54765c649cc'
```

To test that mapping worked, you can send a JSON-RPC curl request to the host port:

```bash
curl --data-binary '{"jsonrpc":"1.0","id":"1","method":"getnetworkinfo","params":[]}' http://foo:qDDZdeQ5vw9XXFeVnXT4PZ--tGN2xNjjR4nrtyszZx0=@127.0.0.1:45339/
```

#### Mainnet

- JSON-RPC/REST: 34229
- P2P: 34230

#### Testnet

- Testnet JSON-RPC: 44229
- P2P: 44230

#### Regtest

- JSON-RPC/REST: 45339
- P2P: 45340


## Docker

This image is officially supported on Docker version 17.09, with support for older versions provided on a best-effort basis.
