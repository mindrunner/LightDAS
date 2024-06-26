# LightDAS
LightDAS is a lighter version of the [Metaplex Digital Asset RPC API](https://github.com/metaplex-foundation/digital-asset-rpc-infrastructure)

**[MUST WATCH DEMO](https://www.loom.com/share/cdea6acd488d4202a16992b45b6e25d1?sid=112a4cc3-4f67-4e1f-a60d-f84c5fad59e2)**  
**[Pitch Deck](https://pitch.com/v/lightdas-gjrunw)**

It allows you to index specific Merkle Trees that you care about. This repository works as a listener and ingester for changes on the Merkle Trees you specify. It does the following:
- Listen on the Merkle Tree address via RPC websockets
- Parse a transaction and deserialize its data, events
- Upsert the Metaplex's DAS database
![LightDAS drawio](https://github.com/WilfredAlmeida/LightDAS/assets/60785452/323da5a6-de11-45a0-bdd2-e5b28d547e71)


### Backfilling and Live Transactions Indexing Process
![lightdas-queue](https://github.com/WilfredAlmeida/LightDAS/assets/60785452/e24fcc69-e5fa-406e-99a3-1cce22815740)
1. LightDAS is started and calls `getSignaturesForAddress` on the Merkle Tree
2. It returns transaction signatures from the latest to the first
3. These are put into the queue one by one via push front. So when step 1 is completed, the queue will have all transaction signatures in an order of first to latest for backfilling
4. LightDAS in parallel to step 1, also calls `logsSubscribe` on the Merkle Trees and listens to live transaction happening on them
5. These are put into the queue one by one via push-back
6. After step 1 is completed and all transaction signatures are present in the queue, a processes task starts in parallel and, processes transactions
7. These processed transactions are inserted into the Metaplex DAS Database
8. The Metaplex DAS API serves DAS requests by interacting with the DAS database

### Reasons we are building LigthDAS
- Running a standard DAS API is expensive and complicated
- It gives you data off all of the NFTs on chain, but do you really need all of it?
- There are select DAS offerings thus creating a monopolistic environment

With LightDAS, you can have your own DAS API without the nft ingester or any other heavy lifting. The components you need to get your DAS running are:
- LightDAS ingester (us)
- DAS API Handler (Metaplex)
- DAS Database (Metaplex)
- Graphite Monitoring (Metaplex)

## Getting started
Follow the steps mentioned below

### Metaplex DAS
- Clone the [Metaplex Digital Asset RPC API](https://github.com/metaplex-foundation/digital-asset-rpc-infrastructure) repo
- You need the `api`, `db`, and `graphite` containers
- Run `docker compose up`. This'll take some time to build and start the containers. Depending on your machine, you can comment out services in the `docker-compose.yaml` if you want them built
- After the build is successful, you can stop all other containers except the ones mentioned above
- Then configure and run LightDAS

### LightDAS
- Clone the repo
- Add environment variables:
  - `RPC_URL`: RPC needs to support websocket functions. We've built using [Quicknode](https://www.quicknode.com/?via=aayush)
  - `WS_URL`: RPC websocket URL
  - `DATABASE_URL`: Default is `postgres://solana:solana@localhost:5432/solana`, use this unless you changed anything
- Execute `cargo run`
- This will download and compile the code with all needed dependencies. Grab a coffee this takes a while
- Once running, you'll see the logs of the tasks being performed
- Under heavy loads, we have faced RPC rate limits
- RPC Costs per NFT Mint:
  - Quicknode:
    - `logsSubscribe`: 50 credits
    - `getTransaction`: 50 credits
- Overall, each NFT mint will cost you 100 RPC credits

**Currently LightDAS supports only Compressed NFTs**:

### Testing
If the program is running without any errors then the database is populated with information on new NFT mints. You can query the RPC API locally. It runs on the default URL `http://localhost:9090/`

### Note
1. Asynchronous backfilling is broken and the logic needs to be redone
2. Backfilling is disabled by default, enable it by uncommenting the block at `src/main.rs:148`

### Support
If you need any help, have any thoughts, or need to get in touch, DM [Wilfred](https://twitter.com/WilfredAlmeida_) on Twitter/X or open an issue.

We have some open [RFCs](https://github.com/WilfredAlmeida/LightDAS/labels/rfc) and need your thoughts  

### Roadmap
The following is our roadmap in decreasing order of priority:  
- Test API responses correctness against standard DAS API responses
- Publish benchmarking results of testing with different RPC providers under various deployment environments
- Test out if LightDAS can work as a full fledged DAS. Since we're watching Merkle trees, we can also watch the Bubblegum program and index all NFT operations.

### The Future of LightDAS
Our vision for LightDAS is to keep it an open-source public good for everyone. We aim to be a compliment to DAS, not to compete against it. Eventually, we would like to streamline the setup process to only need a single binary with minimal dependencies so it's easy for a project to setup a light DAS client to watch a tree and start serving requests. The future decisions for LightDAS will be based on community feedback and discussions.

To keep building LightDAS, we need your support and thoughts. It can be contributions, money/grants, hiring us, providing us with resources, etc. Get in touch.

### Licensing
All code is licensed under the GNU Affero General Public License v3.0 or later.

### Humans at LightDAS
[Wilfred Almeida](https://twitter.com/WilfredAlmeida_)  
[Kartik Soneji](https://github.com/KartikSoneji)
