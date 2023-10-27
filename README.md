<div align="center">
  <h1>Beerus</h1>
    <img src="book/images/beerus.png" height="250">
  <br />
  <br />
  <a href="https://github.com/keep-starknet-strange/beerus/issues/new?assignees=&labels=bug&template=01_BUG_REPORT.md&title=bug%3A+">Report a Bug</a>
  -
  <a href="https://github.com/keep-starknet-strange/beerus/issues/new?assignees=&labels=enhancement&template=02_FEATURE_REQUEST.md&title=feat%3A+">Request a Feature</a>
  -
  <a href="https://github.com/keep-starknet-strange/beerus/discussions">Ask a Question</a>
</div>

<div align="center">
<br />

[![GitHub Workflow Status](https://github.com/keep-starknet-strange/beerus/actions/workflows/check.yml/badge.svg)](https://github.com/keep-starknet-strange/beerus/actions/workflows/check.yml)
[![Project license](https://img.shields.io/github/license/keep-starknet-strange/beerus.svg?style=flat-square)](LICENSE)
[![Pull Requests welcome](https://img.shields.io/badge/PRs-welcome-ff69b4.svg?style=flat-square)](https://github.com/keep-starknet-strange/beerus/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)

</div>

## About

Beerus is a Starknet Light Client inspired by and using
[helios](https://github.com/a16z/helios/). The goal is to provide a simple and
easy to use client to query Starknet state and interact with contracts.

See the [Beerus Book](book/README.md) for more info.

## Getting Started

### Config

Beerus relies on TWO untrusted RPC endpoints, as these are untrusted they will
typically not be nodes run on your local host or your local network. These 
untrusted RPC providers musts adhere to both the l1 `eth_getProof` endpoint
as well as the l2 `pathfinder_getProof` endpoint. For this we recommend using
[Alchemy](https://www.alchemy.com) as your untrusted node provider. 

NOTE: we rely on helios for both valid checkpoint values and consensus rpc urls.

| Field   | Values | Description |
| ----------- | ----------- | ----------- |
| network | MAINNET or GOERLI | network to query |
| eth_execution_rpc | https://eth-mainnet.g.alchemy.com/v2/YOURAPIKEY | untrusted l1 node provider url |
| starknet_rpc | https://starknet-mainnet.g.alchemy.com/v2/YOURAPIKEY | untrusted l2 node provider url |
| data_dir | tmp | `OPTIONAL` location to store both l1 and l2 data |
| poll_secs | 5 | `OPTIONAL` seconds to wait for querying sn state |
| rpc_addr | 127.0.0.1:3030 | `OPTIONAL` local address to listen for rpc reqs |
| fee_token_addr | 0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7 | `OPTIONAL` fee token to check for `getBalance` |

Direct the beerus cli to your config file as follows:

```bash
cargo build
./target/debug/beerus -c examples/conf/beerus.json

# or

cargo run -p beerus-cli -- -c examples/conf/beerus.toml

# wait for server to start
hurl examples/rpc/starknet_provenStateRoot.hurl
```

### Examples

```bash
cargo run -p beerus-core --example basic
```
Using config from `.toml` file
```bash
BEERUS_CONFIG=path/to/config.toml cargo run -p beerus-core --example basic
```

### Using Beerus as a Library

Beerus can be imported into any Rust project.

```rust
use beerus_core::{config::Config, lightclient::beerus::BeerusLightClient};
use env_logger::Env;
use eyre::Result;

#[tokio::main]
async fn main() -> Result<()> {
  env_logger::Builder::from_env(Env::default().default_filter_or("info")).init();
  let config = Config::from_env();

  let mut beerus = BeerusLightClient::new(config.clone()).await?;
  beerus.start().await?;

  let current_starknet_block = beerus.starknet_lightclient.block_number().await?;
  println!("{:?}", current_starknet_block);

  let current_ethereum_block = beerus
          .ethereum_lightclient
          .lock()
          .await
          .get_block_number()
          .await?;
  println!("{:?}", current_ethereum_block);
  Ok(())
}
```

## Development

#### Build

```bash
cargo build --all --release
```

#### Test

```bash
cargo test --all
```

##### Beerus JS(wasm demo)

Dependencies:

- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- [CORS bypass](https://github.com/garmeeh/local-cors-proxy/blob/master/README.md)
- local pathfinder node at `http://localhost:9545`
- execution env var - `ETHEREUM_EXECUTION_RPC_URL`

```bash
cd crates/beerus-js

# install node deps
npm i

# build webpack & wasm modules
npm run build

# run example
./run.sh

# navigate browser to http://localhost:8080
# open developer console
```

## Endpoint support

Here are all the endpoints supported by Beerus in tag v0.2.0

*Starknet endpoints* (20) (in compliance with [Starknet specs](https://playground.open-rpc.org/?uiSchema%5BappBar%5D%5Bui:splitView%5D=false&schemaUrl=https://raw.githubusercontent.com/starkware-libs/starknet-specs/master/api/starknet_api_openrpc.json&uiSchema%5BappBar%5D%5Bui:input%5D=false&uiSchema%5BappBar%5D%5Bui:darkMode%5D=true&uiSchema%5BappBar%5D%5Bui:examplesDropdown%5D=false)):

| Endpoint                                   | Supported          |
| :----------------------------------------- | :----------------- |
| `starknet_getBlockWithTxHashes`            | :white_check_mark: |
| `starknet_getBlockWithTxs`                 | :white_check_mark: |
| `starknet_getStateUpdate`                  | :white_check_mark: |
| `starknet_getStorageAt`                    | :white_check_mark: |
| `starknet_getTransactionByHash`            | :white_check_mark: |
| `starknet_getTransactionByBlockIdAndIndex` | :white_check_mark: |
| `starknet_getTransactionReceipt`           | :white_check_mark: |
| `starknet_getClass`                        | :white_check_mark: |
| `starknet_getClassHashAt`                  | :white_check_mark: |
| `starknet_getClassAt`                      | :white_check_mark: |
| `starknet_getBlockTransactionCount`        | :white_check_mark: |
| `starknet_call`                            | :white_check_mark: |
| `starknet_estimateFee`                     | :white_check_mark: |
| `starknet_blockNumber`                     | :white_check_mark: |
| `starknet_blockHashAndNumber`              | :white_check_mark: |
| `starknet_chainId`                         | :white_check_mark: |
| `starknet_pendingTransactions`             | :white_check_mark: |
| `starknet_syncing`                         | :white_check_mark: |
| `starknet_provenStateRoot`                 | :white_check_mark: |
| `starknet_provenBlockNumber`               | :white_check_mark: |
| `starknet_getBalance`                      | :white_check_mark: |
| `starknet_syncing`                         | :white_check_mark: |
| `starknet_getEvents`                       | :x:                |
| `starknet_getNonce`                        | :white_check_mark: |
| `starknet_addDeclareTransaction`           | :x:                |
| `starknet_addDeployAccountTransaction`     | :x:                |
| `starknet_getContractStorageProof`         | :x:                |
| `starknet_addInvokeTransaction`            | :x:                |

## Work in progress

See the [open issues](https://github.com/keep-starknet-strange/beerus/issues) for
a list of proposed features (and known issues).

- [Top Feature Requests](https://github.com/keep-starknet-strange/beerus/issues?q=label%3Aenhancement+is%3Aopen+sort%3Areactions-%2B1-desc)
  (Add your votes using the 👍 reaction)
- [Top Bugs](https://github.com/keep-starknet-strange/beerus/issues?q=is%3Aissue+is%3Aopen+label%3Abug+sort%3Areactions-%2B1-desc)
  (Add your votes using the 👍 reaction)
- [Newest Bugs](https://github.com/keep-starknet-strange/beerus/issues?q=is%3Aopen+is%3Aissue+label%3Abug)

## Support

Reach out to the maintainer at one of the following places:

- [GitHub Discussions](https://github.com/keep-starknet-strange/beerus/discussions)
- Contact options listed on
  [this GitHub profile](https://github.com/keep-starknet-strange)

## Project assistance

If you want to say **thank you** or/and support active development of Beerus:

- Add a [GitHub Star](https://github.com/keep-starknet-strange/beerus) to the
  project.
- Tweet about the Beerus.
- Write interesting articles about the project on [Dev.to](https://dev.to/),
  [Medium](https://medium.com/) or your personal blog.

Together, we can make Beerus **better**!

## Contributing

First off, thanks for taking the time to contribute! Contributions are what make
the open-source community such an amazing place to learn, inspire, and create.
Any contributions you make will benefit everybody else and are **greatly
appreciated**.

Please read [our contribution guidelines](docs/CONTRIBUTING.md), and thank you
for being involved!

## Security

Beerus follows good practices of security, but 100% security cannot be assured.
Beerus is provided **"as is"** without any **warranty**. Use at your own risk.

_For more information and to report security issues, please refer to our
[security documentation](docs/SECURITY.md).

## Acknowledgements

- Huge props to A16z for their work on
  [helios](https://github.com/a16z/helios/).

## Contributors ✨

[The contributors page](https://github.com/keep-starknet-strange/beerus/contributors).

Thanks goes to these wonderful people
([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/abdelhamidbakhta"><img src="https://avatars.githubusercontent.com/u/45264458?v=4?s=100" width="100px;" alt="Abdel @ StarkWare"/><br /><sub><b>Abdel @ StarkWare</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=abdelhamidbakhta" title="Tests">⚠️</a> <a href="https://github.com/keep-starknet-strange/beerus/commits?author=abdelhamidbakhta" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/LucasLvy"><img src="https://avatars.githubusercontent.com/u/70894690?v=4?s=100" width="100px;" alt="Lucas @ StarkWare"/><br /><sub><b>Lucas @ StarkWare</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=LucasLvy" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/danilowhk"><img src="https://avatars.githubusercontent.com/u/12735159?v=4?s=100" width="100px;" alt="danilowhk"/><br /><sub><b>danilowhk</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=danilowhk" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://www.linkedin.com/in/clementwalter"><img src="https://avatars.githubusercontent.com/u/18620296?v=4?s=100" width="100px;" alt="Clément Walter"/><br /><sub><b>Clément Walter</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=ClementWalter" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Eikix"><img src="https://avatars.githubusercontent.com/u/66871571?v=4?s=100" width="100px;" alt="Elias Tazartes"/><br /><sub><b>Elias Tazartes</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=Eikix" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/drspacemn"><img src="https://avatars.githubusercontent.com/u/16685321?v=4?s=100" width="100px;" alt="drspacemn"/><br /><sub><b>drspacemn</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=drspacemn" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/msaug"><img src="https://avatars.githubusercontent.com/u/60658558?v=4?s=100" width="100px;" alt="Mathieu"/><br /><sub><b>Mathieu</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=msaug" title="Code">💻</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/hurrikaanig"><img src="https://avatars.githubusercontent.com/u/37303126?v=4?s=100" width="100px;" alt="TurcFort07"/><br /><sub><b>TurcFort07</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=hurrikaanig" title="Documentation">📖</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/florian-bellotti"><img src="https://avatars.githubusercontent.com/u/7861901?v=4?s=100" width="100px;" alt="Florian Bellotti"/><br /><sub><b>Florian Bellotti</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=florian-bellotti" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/bbrandtom"><img src="https://avatars.githubusercontent.com/u/45038918?v=4?s=100" width="100px;" alt="Tom Brand"/><br /><sub><b>Tom Brand</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=bbrandtom" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/ftupas"><img src="https://avatars.githubusercontent.com/u/35031356?v=4?s=100" width="100px;" alt="ftupas"/><br /><sub><b>ftupas</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=ftupas" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/pscott"><img src="https://avatars.githubusercontent.com/u/30843220?v=4?s=100" width="100px;" alt="pscott"/><br /><sub><b>pscott</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=pscott" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/robinstraub"><img src="https://avatars.githubusercontent.com/u/17799181?v=4?s=100" width="100px;" alt="Robin Straub"/><br /><sub><b>Robin Straub</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=robinstraub" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/fkrause98"><img src="https://avatars.githubusercontent.com/u/56402156?v=4?s=100" width="100px;" alt="Francisco Krause Arnim"/><br /><sub><b>Francisco Krause Arnim</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=fkrause98" title="Documentation">📖</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/joshualyguessennd"><img src="https://avatars.githubusercontent.com/u/75019812?v=4?s=100" width="100px;" alt="joshualyguessennd"/><br /><sub><b>joshualyguessennd</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=joshualyguessennd" title="Documentation">📖</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/dubzn"><img src="https://avatars.githubusercontent.com/u/58611754?v=4?s=100" width="100px;" alt="Santiago Galván (Dub)"/><br /><sub><b>Santiago Galván (Dub)</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=dubzn" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/chirag-bgh"><img src="https://avatars.githubusercontent.com/u/76247491?v=4?s=100" width="100px;" alt="chirag-bgh"/><br /><sub><b>chirag-bgh</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=chirag-bgh" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/greged93"><img src="https://avatars.githubusercontent.com/u/82421016?v=4?s=100" width="100px;" alt="greged93"/><br /><sub><b>greged93</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=greged93" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/bigherc18"><img src="https://avatars.githubusercontent.com/u/126212764?v=4?s=100" width="100px;" alt="bigherc18"/><br /><sub><b>bigherc18</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=bigherc18" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Kelvyne"><img src="https://avatars.githubusercontent.com/u/8125532?v=4?s=100" width="100px;" alt="Lakhdar Slaim"/><br /><sub><b>Lakhdar Slaim</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=Kelvyne" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://linktr.ee/lndavis"><img src="https://avatars.githubusercontent.com/u/40670744?v=4?s=100" width="100px;" alt="Lance N. Davis"/><br /><sub><b>Lance N. Davis</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=lancenonce" title="Code">💻</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/tinoh9"><img src="https://avatars.githubusercontent.com/u/97869487?v=4?s=100" width="100px;" alt="Tino Huynh"/><br /><sub><b>Tino Huynh</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=tinoh9" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/irisdv"><img src="https://avatars.githubusercontent.com/u/8224462?v=4?s=100" width="100px;" alt="Iris"/><br /><sub><b>Iris</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=irisdv" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Aragar199"><img src="https://avatars.githubusercontent.com/u/14187644?v=4?s=100" width="100px;" alt="Alex Ponce"/><br /><sub><b>Alex Ponce</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=Aragar199" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/glihm"><img src="https://avatars.githubusercontent.com/u/7962849?v=4?s=100" width="100px;" alt="glihm"/><br /><sub><b>glihm</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=glihm" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/phklive"><img src="https://avatars.githubusercontent.com/u/42912740?v=4?s=100" width="100px;" alt="Paul-Henry Kajfasz"/><br /><sub><b>Paul-Henry Kajfasz</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=phklive" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/dpinones"><img src="https://avatars.githubusercontent.com/u/30808181?v=4?s=100" width="100px;" alt="Damián Piñones"/><br /><sub><b>Damián Piñones</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=dpinones" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/betacodd"><img src="https://avatars.githubusercontent.com/u/97968794?v=4?s=100" width="100px;" alt="Betacodd"/><br /><sub><b>Betacodd</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=betacodd" title="Code">💻</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Godspower-Eze"><img src="https://avatars.githubusercontent.com/u/61994334?v=4?s=100" width="100px;" alt="Godspower-Eze"/><br /><sub><b>Godspower-eze</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=Godspower-Eze" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/somthn0somthn"><img src="https://avatars.githubusercontent.com/u/41335589?v=4?s=100" width="100px;" alt="somthn0somthn"/><br /><sub><b>somthn0somthn</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=somthn0somthn" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/tonypony220"><img src="https://avatars.githubusercontent.com/u/61715244?v=4?s=100" width="100px;" alt="tonypony220"/><br /><sub><b>tonypony220</b></sub></a><br /><a href="https://github.com/keep-starknet-strange/beerus/commits?author=tonypony220" title="Code">💻</a></td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td align="center" size="13px" colspan="7">
        <img src="https://raw.githubusercontent.com/all-contributors/all-contributors-cli/1b8533af435da9854653492b1327a23a4dbd0a10/assets/logo-small.svg">
          <a href="https://all-contributors.js.org/docs/en/bot/usage">Add your contributions</a>
        </img>
      </td>
    </tr>
  </tfoot>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the
[all-contributors](https://github.com/all-contributors/all-contributors)
specification. Contributions of any kind welcome!
