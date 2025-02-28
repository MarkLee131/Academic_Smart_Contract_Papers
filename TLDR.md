I will stop to share my summary here, please update my future summary on my LinkedIn posts page:[https://www.linkedin.com/in/zheyuan-he-7a1207338/recent-activity/all/](https://www.linkedin.com/in/zheyuan-he-7a1207338/recent-activity/all/). 

-----------

# 25_2_25

## NDSS25 | [The Forking Way: When TEEs Meet Consensus](https://www.ndss-symposium.org/wp-content/uploads/2025-1934-paper.pdf)

## Motivation
The authors first summarize the application and safety of Trusted Execution Environments (TEEs) on blockchain consensus mechanisms.

## Background
TEE is a hardware-isolated technology designed to ensure that code execution is secure, confidential, and maintains integrity. Ideally, decoupling heavy on-chain computations to an off-chain TEE environment (termed enclave) will significantly improve the performance of the blockchain.

## Taxonomy
The authors first collect 28 TEE-based blockchain platforms from [this GitHub repository](https://github.com/Maxul/Awesome-SGX-Open-Source?tab=readme-ov-file#blockchains), and categorize them into four types:
1. **TEE-based Smart Contracts**
2. **TEE-based Consensus Protocols**
3. **TEE-based Layer 2 Solutions**
4. **TEE-based Blockchain Applications**

## TEE Application Scenarios
The authors further classify TEE applications into six scenarios:
1. **Stateless Enclaves**: Stateless enclaves are designed not to maintain any persistent state between executions, relying on the untrusted host to provide necessary state information each time the enclave is invoked.
2. **Ephemeral Identities**: Ephemeral identities generate a unique identifier for each enclave instance upon restart, which prevents forging the off-chain enclave.
3. **Fixed Set of Clients**: A fixed set of clients involves limiting interactions with the smart contract to a predefined group of trusted clients.
4. **Serializing State (Transaction Replay)**: In transaction replay, the enclave recovers its state by reprocessing all transactions from the genesis block to the current block.
5. **Serializing State (Timestamp)**: Timestamping involves storing the state along with metadata such as the block height, hash, and timestamp. When responding to client queries, the enclave includes this timestamp to ensure the freshness of the state.
6. **Serializing State (Ledger Storage)**: In ledger storage, the enclave directly stores its state on the blockchain ledger.

## Attack Types
There are two primary types of attacks for TEE-based blockchains:
1. **Cloning Attack**: A cloning attack occurs when an adversary creates multiple instances of a TEE and uses the most favorable output to deceive the blockchain node.
2. **Fork Attack**: A fork attack happens when an attacker exploits blockchain forks to present outdated state information, misleading blockchain nodes about the current valid state.

## Evaluation
The authors conduct a Proof of Concept (PoC) experiment in a private chain environment on three TEE-based blockchains: Ten, Phala, and Secret Network. To demonstrate the attack, the authors launch two enclaves, isolate one of them, and then redirect on-chain requests to the isolated enclave, thereby forking the chain. Specifically:
- They start a Ten enclave that completes enrollment, obtains the master seed, and seals cryptographic keys to disk.
- Both enclave instances have access to the state sealed by the first enclave.
- The adversary receives a new block from an L1 node and feeds it to both clones.
- The enclaves generate random nonces $ N $ and $ N' $, respectively, and include them in the proposed rollup.
- The adversary selects the rollup with the lowest nonce and submits it to the L1 layer.

  ----------------

# 25_2_14
## CCS24 | [DoubleUp Roll: Double-spending in Arbitrum by Rolling It Back](https://0xzhiyuan.github.io/files/DoubleUp_Roll__Final_Version_.pdf)
* Motivation: This paper proposes various state rollback attack strategies in a cross-chain environment, specifically targeting L2 platforms such as Arbitrum and Optimism. The adversary can exploit these strategies to make significant profits by reverting deposit transactions on L2 chain while successfully withdrawing funds in L1 chain.
* Background: 
  * Every transaction in L2 needs to be committed to L1 to finalize its state. This commitment process involves batching transactions and submitting them to L1 within a specified time window (V). 
  * There are two types of finalization: soft-finalization and hard-finalization. Soft-finalization refers to a temporary state where a transaction is processed and recorded on L2 but has not yet been confirmed on L1; it remains reversible if any issues are detected. Hard-finalization, on the other hand, occurs when a transaction has been fully committed to L1 and becomes immutable, ensuring that it cannot be altered or removed. 
  * Many decentralized applications (DApps) only require soft-finalization because they prioritize fast confirmation times and lower costs over the absolute finality provided by hard-finalization. This characteristic makes DApps more susceptible to certain types of attacks, such as those described in this paper, which exploit the vulnerabilities during the soft-finalization phase.
* Basic Idea
  * **Construct Exploited Transactions**: The adversary creates a series of exploited transactions designed to cause delays or manipulate the blockchain.
  * **Send Deposit Transaction on L2**: The adversary sends a deposit transaction on L2, which gets soft-finalized but faces delays in hard-finalizing on L1.
  * **Withdrawal on L1**: The adversary attempts to withdraw funds from L1. Due to the exploited transactions, the deposit transaction on L2 will be reverted, allowing the withdrawal on L1 to proceed normally, thus enabling the adversary to profit.
* Exploit Method 1: Delay Attack
  * **Spam Transactions on L2**: The adversary constructs spam transactions containing large amounts of non-compressible data.
  * **Deposit Transaction**: Simultaneously, the adversary sends a deposit transaction on L2, which gets soft-finalized.
  * **Delay Finalization**: The spam transactions create significant delays, preventing the deposit transaction from being finalized on L1 within the allowed time window V.
  * **State Rollback**: Once the delay V exceeds the threshold, the L2 transactions are reverted due to the time boundaries mechanism.
* Exploit Method 2: Force Inclusion Attack
  * **Spam Transactions on L2**: Similar to Method 1, the adversary sends spam transactions to create delays.
  * **Deposit Transaction**: The adversary sends a deposit transaction on L2.
  * **Force Commit Withdrawal**: The adversary uses a special mechanism (e.g., `ForceInclusion` function of Liveness-Preservation Mechanism) to commit the withdrawal transaction on L1 before the deposit transaction is finalized.
  * **Revert Deposit**: Since the withdrawal transaction on L1 is confirmed earlier than the deposit transaction, the deposit transaction on L2 is reverted.
* Exploit Method 3: Size-Based Attack
  * **Large Deposit Transaction**: The adversary constructs a deposit transaction with an exceptionally large size.
  * **Compression Without Size Check**: Before using the transaction, L2 blockchain compresses it without performing any size-bound checks.
  * **Decompression Failure**: When L2 tries to decompress the transaction, it exceeds the size-bound limitation, causing the deposit transaction to be reverted.
* Challenge: The primary challenge for the adversary is the high cost associated with spamming transactions to create delays.
* Solution: The authors discovered a calculation flaw in the pricing mechanism where surplus handling caused an implicit type conversion, treating negative surplus values as positive. This error significantly amplified the price adjustment, allowing attackers to manipulate the posting unit price to near zero and drastically reduce their attack costs.
* Evaluation:
  * **Transaction Delay**: The delay attack can postpone the transaction by approximately 217 seconds.
  * **Cost Optimization**: The cost optimization method reduces the attack cost by up to 92%.
* Bounty: The identified vulnerabilities were acknowledged and addressed by the relevant platforms, significantly mitigating risks to billions of dollars in assets. Researchers responsible for uncovering these issues were awarded a substantial bug bounty totaling half a million dollars. 



# 24_12_31
## NDSS24 | [Content Censorship in the InterPlanetary File System](https://www.ndss-symposium.org/wp-content/uploads/2024-153-paper.pdf)
* Motivation: The authors propose and evaluate a content censorship attack on IPFS (InterPlanetary File System), which aims to prevent users from accessing resources stored within the network.
* Background:IPFS is a decentralized peer-to-peer hypermedia protocol designed to make the web faster, safer, and more open. It functions as a content delivery network where each node maintains a distributed hash table (DHT) that helps locate other nodes and files based on their unique identifiers. The distance between nodes is calculated using XOR distance on hash values derived from their public keys. Nodes are organized so that each one keeps track of its closest neighbors in the DHT, facilitating efficient resource discovery.
* Attack Design: The attack proposed by the authors employs a Sybil attack strategy, where an adversary spawns a some malicious nodes to overwhelm honest nodes. Specifically:
  * **Brute Force Generation**: The attacker brute-forces the generation of nodes that are close to the target node's identifier in the DHT space.
  * **Malicious Node Deployment**: By deploying multiple malicious nodes with carefully chosen IDs, the attacker can position them as the closest neighbors to the target node, effectively replacing honest communication nodes.
  * **Censorship**: This allows the attacker to intercept and block all resource requests made by the target node, thereby censoring access to certain content.
* Attack Detection:
To detect such attacks, the authors introduce a method that involves sampling normal nodes and calculating distances using the DHT. They compare these distances against the set of closest nodes maintained by a target node using the Kullback-Leibler (KL) divergence algorithm:
  * **Sampling and Calculation**: Sample normal nodes and calculate distances using the DHT.
  * **Comparison**: Compare these distances against the closest nodes set using KL divergence.
  * **KL Divergence**: Measures how one probability distribution diverges from another expected distribution. In this case, it quantifies the difference between the observed distribution of common prefix lengths (CPLs) from sampled nodes and the theoretical geometric distribution expected under honest conditions. A significant deviation indicates potential censorship activity.
* Attack Mitigation: For mitigating the attack, the authors suggest expanding the scale of the closest nodes set beyond the typical k nearest neighbors from region-based DHT:
  * **Increase Scale**: By increasing the number of nodes involved in queries and record storage, the cost of launching an effective Sybil attack rises significantly.
  * **Resilience**: Honest nodes become harder to isolate, ensuring that provider records can still reach at least some legitimate peers even when facing a substantial number of Sybil nodes.
  * **Region-Based Queries**: Employ region-based DHT queries to further enhance resilience against censorship.
* Evaluation: The evaluation demonstrates the effectiveness of both the attack and the detection/mitigation strategies:
  * **Attack Success**: Using 45 Sybil nodes, the attack successfully blocked 99% of users' content requests, showcasing   * **Detection Accuracy**: The detection method achieved a low false negative rate of 0.81% and a modest false positive rate of 4.4%, indicating reliable identification of attacks.
  * **Mitigation Effectiveness**: The mitigation strategy ensured 100% successful content discovery despite the presence of Sybil nodes, highlighting its robustness and practicality.
* Comments: The authors comprehensively detail the entire lifecycle of the attack, encompassing its design, detection, and mitigation. It is particularly insightful to include both detection and mitigation strategies in a security paper. This approach not only enhances the contribution of their work but also adheres to ethical considerations by providing solutions to counteract the identified vulnerabilities.





# 24_12_15
## CCS24 | fAmulet: Finding Finalization Failure Bugs in Polygon zkRollup
* Motivation: This study aims to identify bugs in the Polygon network that can disrupt the interoperability between Layer 1 (L1) and Layer 2 (L2) chains, focusing specifically on issues affecting the finalization processes.
* Background: In Polygon, transactions undergo a two-step finalization process: 1. **Fast Finalization**: Transactions are first ordered by the Sequencer component and then submitted to the L1 chain (e.g., Ethereum). 2.**Hard Finalization**: Transactions are sent to the Executor component for execution, state transition, and generation of zero-knowledge proofs (ZK proofs). The Aggregator component commits the ZK proofs to the L1 chain.Additionally, an emergency mechanism called **forced batch** allows transactions to be committed directly to the Executor component without going through the Sequencer, serving as a fallback pathway.
* Threat Model: The authors define two types of bugs based on their impact on the finalization processes: 1. **Fast Finalization Failure Bugs (FFFBs)**: Affect the commitment of transactions to the L1 chain. 2.**Hard Finalization Failure Bugs (HFFBs)**: Affect the commitment of ZK proofs to the L2 chains.
* Approach：To detect these FFFBs and HFFBs, the authors developed a fuzzing tool named **fAmulet**. This tool models the states of key Polygon components—the Sequencer, Executor, and Aggregator—and uses mutation testing and feedback-driven coverage analysis to uncover bugs. By mutating test cases and analyzing resulting states, fAmulet identifies vulnerabilities that could compromise the finalization processes.
* Evaluation:fAmulet successfully uncovered 12 zero-day bugs, achieving 20.8% more branch coverage compared to baseline methods. This highlights the effectiveness of the tool in identifying previously undiscovered vulnerabilities.
* Case Study: A case study revealed a bug where a developer mistakenly set the memory bound to `0x10000` instead of the correct value `0x20000`. An attacker exploited the forced batch commit mechanism to access memory address `0x10001`, causing the Executor component to fail processing the transaction and disrupting fast finalization.

# 24_12_03
## ICSE25 | Demystifying and Detecting Cryptographic Defects in Ethereum Smart Contracts
* Motivation: This study aims to identify and mitigate the misuse of cryptographic functions within the deployment of Ethereum smart contracts. With the growing importance of blockchain technology, the security of smart contracts has become critical since once deployed on the blockchain, they are challenging to alter. Therefore, detecting and correcting potential security vulnerabilities is essential for protecting user assets and information.
* Background: Smart contracts often employ cryptographic functions in their access control logic. For instance, precompiled contracts like `ecrecover` are used to verify the `r`, `s`, `v` values provided by users for delegation business. Proper application of these cryptographic primitives is vital for ensuring the safety and reliability of smart contracts.
* Obervation: By collecting information from blogs, audit reports, and GitHub, the authors identified nine types of misuse scenarios involving cryptographic functions. These scenarios include:
  - **Signature Misuse**: Improper verification or parsing of digital signatures.
  - **Merkle Proof Misuse**: Incorrect use of Merkle proofs, potentially allowing attackers to bypass validation mechanisms.
  - **Hash Collisions**: Exploiting weaknesses in hash functions that could lead to collisions.
  - **Weak Randomness**: Using insufficiently random numbers, which can be predicted or influenced by an attacker.
Adversaries can exploit such misuses to circumvent access control mechanisms, leading to unauthorized operations or data leaks.
* Approach: To detect the above nine types of misuses, the authors developed a fuzzing tool named CRYSOL. CRYSOL utilizes historical transactions as seeds and mutates them based on patterns derived from the nine misuse scenarios. The tool combines transaction replaying and dynamic taint analysis to extract fine-grained crypto-related semantics and employs crypto-specific strategies to guide the test case generation process.
* Evaluation: The authors evaluated CRYSOL on a dataset of 25,745 real-world crypto-related smart contracts. CRYSOL uncovered 5,847 instances of misuse with a precision rate of 95.4%. This evaluation demonstrates CRYSOL's effectiveness in identifying cryptographic defects in smart contracts and highlights the prevalence of such issues in practice.

# 24_11_30
## Sec24 | Speculative Denial-of-Service Attacks in Ethereum
* Motivation: this work proposes three exploit methods to launch the DoS attack for Ethereum's proposer.
* Main ideas: the authors contrast the malicious transactions by crafting the logic of smart contracts, sanctioned addresses, and cross-transaction dependency relations.
* Attack1: the author constructed a type of conditional resource exhaust attack. Concretely, the authors craft the transaction interacted with an exploit contract. In the exploit contract, the author uses some block environment parameters (e.g., specific block number or block miner) as a condition to trigger some heavy I/O logic (i.e., payload) to decrease the proposer's efficiency. The condition of specific block numbers and block miners in the exploited contract ensures the time and target (i.e., validator) of the attack. 
* The cost-saving trick of attack1: The author adopts an interesting trick to save the cost of attack, which makes the exploit contract interact with the sanctioned addresses (The addresses that are banned by powerful institutions). If the proposer wants to meet compliance requirements, they will drop the malicious transactions after executing the payload. 
* Attack2: the author constructs another attack by constructing a transactions exploit chain. The first transaction in the exploit chains will execute some operations to nullify subsequent transactions (e.g., transferring all the funding of the account related to the rest of the transactions). By doing so, the attack can evict the validated transaction in the memory pool.
* Exploit: The author exploits the above two methods to attack the proposer in the MEV system. The attack2 first evicts the validated transaction in the memory pool, and attack1 will be included as the first transaction in tranaction2's export chains.  Finally, the proposer cannot package any validated transaction in block.
* Evaluation: The authors conducted experiments on a testnet, simulating attacks with specific parameters, showing up to 96% block sync slowdown and 80% mempool clogging.


# 24_11_1
### Sec24 | Max Attestation Matters: Making Honest Parties Lose Their Incentives in Ethereum PoS
* Motivation: this work proposes a new method to attack the honest stakers in the Ethereum Poof-of-Stake (PoS) mechanism, which causes the stakers to lose part of their staking funding.
* Background: in Ethereum's PoS, every block will be proposed in a slot and every 32 slots will constitute a epoch. To keep the consensus and security of the chain, PoS mechanism constructs a type of actors called validators to propose and validate every block. To become the validators, the user must stake 32 eth on the Ethereum. To finality and check every epoch, every validator will vote on the last epoch (source) to the current epoch (target). The validator who doesn't vote will get the penalty.
* Threat models: the attacker must be the proposer of the first slot in an epoch and control 29.8% of total staking funding.
* Core idea: the attacker as the proposer uninterruptedly withholds the block to disturb the visibility (a.k.a view) of the honest stakers. At this moment, the honest stakers will vote for attestation with the wrong source and target (as the attacker withholds the block, the chain in honest stakers is discrete). The Ethereum will discard the wrong attestation and the honest staker will get the penalty.
* Poof-of-Conception: the authors simulate an environment with 1000 validators running Prysm. The result shows if the adversary controls a 29.6% stake, all honest validators lose their incentives.

# 24_10_22
### Infocom24 | DEthna: Accurate Ethereum Network Topology Discovery with Marked Transactions
* Motivation: This work aims to discover the network topology in EL layers.
* Background: The Ethereum EL layer is designed to conceal its network topology for security purposes. Every node in the EL network can only connect a few nodes.
* Current research: The current works need to consume large resources (transaction numbers) with inefficiency.
* Main Idea: The author proposes a technology named marked transaction. concretely, the author first sends three transactions (tx1, tx2, tx3) with the same notices and different gas prices to three nodes (n1,n2,n3). By constructing the nonce and gas price, tx1 can be replaced by tx2, but tx1 and tx2 can't be replaced by tx3. The probing node controlled by the author connects to n1. When the probe node receives tx2 from n1 (replacement occurred), it means n1 and n2 are connected.
* Implementation: The authors achieve their ideas by developing a tool named DEthna based on Geth. DEthna adopts the distributed design to connect and probe more network nodes in EL's network.
* Evaluation: The authors evaluate the DEthna in the Ethereum's Goerli testnet. DEthna can find more than 2.5 times more nodes than the baseline (i.e., Toposhot).
* Observation: The nodes in Goerli testnet establish 16 outbound and 34 inbound connections on average, which will weaken the network's robustness. Every message will take three hops between the blockchain nodes.


# 24_9_18
### CCS24 | TokenScout: Early Detection of Ethereum Scam Tokens via Temporal Graph Learning
* Motivation: This work aims to detect token scams (i.e., Rugpull, Honeypot, and Ponzi) before they happen.
* Prior work: The code analysis of the token contracts cannot cover all scams (e.g., Rugpull wields the normal contract logic), and transaction flow analysis cannot detect the scam that never happens.
* Approach: The authors adopt a neural network to capture the subtle signs before the scam happens. First, they designed a Dynamic Temporal Attributed Multigraph (DTAM) to represent the token transfer flows. Then, the authors construct a Token-Flow Graph Neural Network (TF-GNN) to learn scam features from DTAM. Then, they cluster(refine) the TF-GNN by clustering different scams in embedded space with distinct patterns. They deploy their tool named TokenScout.
* Evaluation: The author detected 214,084 smart contracts and labeled them over 800 human hours. The evaluation results demonstrate that TokenScout achieves a precision of 98.03%, recall of 97.47%, F1 score of 97.75%, and balanced accuracy (BAC) of 98.41% in early-detecting scam tokens.

# 24_5_29
### ICSE24 | SCVHunter: Smart Contract Vulnerability Detection Based on Heterogeneous Graph Attention Network
* Motivation: This work aims to detect the flaws of smart contracts at the level of source code.
* Main Idea: The author converts the contract to a heterogeneous semantic graph and detects flaws in it by machine learning technology.
* Challenge: How do we extract the semantic information from the contract?
* Solution: The author designs a IR, CGIR, to extract to captrue the contract and data information.
* Approach: The author achieves their ideas by developing SCVHunter. SCVHunter converts the source code to CGIR and generates the heterogeneous semantic graph from it. Then, SCVHunter adopts a graph neural network to detect flaws of contracts.
* Evaluation: The author evaluates the SCVHunter on four vulnerabilities: vulnerability, reentrancy, block
info dependency, nested call, and transaction state dependency. It achieves accuracies of 93.72%, 91.07%, 85.41%, and 87.37% for these vulnerabilities.


# 24_5_22
### SP24 | POMABuster: Detecting Price Oracle Manipulation Attacks in Decentralized Finance
* Motivation: This work aims to detect the price oracle manipulation attacks (POMA) in the defi ecosystem.
* Background: POMA indicates the adversary manipulates the coin price to gain profits by arbitrage.
* Prior work: DefiRanger can only detect POMA within a single transaction, but POMA can be exploited by several transactions.
* Approach: The author developed a POMABuster to detect POMA. First, POMABuster filters the transactions by only focusing on high-value crypto assets to reduce search space. Then, PoMABuster detects the POMA based on the rules derived from the author, which are generated by the SEC (in the conventional stock market), including wash sales and market domination.
* Evaluation: The authors evaluate POMABuster on two years of blockchain history and the audit reports from Code4rena. POMA achieves 6.5X more POMA than the prior work, with 1% false positives and 0% false negatives.


# 24_5_20
### SP24 | Large-Scale Study of Vulnerability Scanners for Ethereum Smart Contracts
* Motivation: The author wonders to evaluate the detection capability of contract vulnerability tools.
* Approach: The author measures 18 vulnerability detectors with four categories (symbolic analysis, static analysis, machine learning, and fuzz), in 8 types of flaws.
* Measure detectors: Slither, SmartCheck, Maian, Oyente, Artemis, Osiris, Securify2, Mythril, TeEther, ConFuzzius, Smartian, sFuzz, GNNSCVulDetector, MANDO-GURU, Vandal, Maian, Oyente, and Mythril.
* Measure flaws: Suicide, Reentrancy, Transaction Order Dependency (ToD), Arithmetic Bugs, Usage of txOrigin, Time Dependency, Locked Ether, and DelegateCall.
* Dataset: The authors collect 77,219 source codes, 4 million bytecodes, 14,000 manually annotated, and 373 smart contracts verified through audits.
* Insight1: The tools have different results for the flaws, suggesting that combining several tools won't be very useful in practice.
* Insight2: The tools have poor performance, as their high false positives and negatives.
* Insight3: The compiler versions and code practices of developers will affect the detection results of these tools, which highlight the directions of these tools.

# 24_4_17
### SP24 | Formal Model-Driven Analysis of Resilience of GossipSub to Attacks from Misbehaving Peers
* Motivation: The authors concentrate on whether the GossipSub network can undertake and defend against the attack from its malicious nodes.
* Background: GossipSub manages the set of network nodes with the pub/sub mechanism, which is widely adopted in Ethereum and file coins. GossipSub wields a mechanism named the score function to measure the behaviors of the nodes. The nodes with misbehaves will have a low score, which probably will be dropped.
* Idea: The author combines the document specifications of GossipSub and the consultant from developers to formal verify the GossipSub network.
* Design: The author first defined four security properties of the GossipSub network, (i) If a peer's performance for some topic is continuously non-positive, then, eventually, the peer's score will be non-positive. (ii) When a peer misbehaves, its score decreases. (iii) When a peer behaves, its score does not decrease. (iv) Peers are scored fairly: if they appear to behave identically, they are given identical scores. Then, the authors adopt automatic theorem proving to determine whether configurations (i.g., size, cap, and topology) of GossipSub invalides the security properties.
* Implement: The authors use ACL2s as the theorem prover, and construct the model based on the golang GossipSub.
* Evaluation: The authors uncover the configurations of File coin can defend against the attack from misbehaving peers, but Ethereum can't.

# 24_3_19
### CCS23 | Uncle Maker: (Time)Stamping Out The Competition in Ethereum
* Motivation: The authors focus on the security of Proof of Work (POW) mechanism of Ethereum.
* Background: The tip of the blockchain can be determined by difficulty, and difficulty can be determined by timestamp.
* Idea: The adversary miner manipulates the timestmap of block to decrease the difficulty of the fork chain. The fork chain with higher difficulty mined by adversary will be the tip of current chain, and make more profits for the adversary miner.
* Observation: The authors uncover F2pool adopting such an attack to gain about 3 million USD from March 8th, 2021, to September 15th, 2022.
* Mitigation: The authors propose two mitigations, including increasing the minimal difficulty and rejecting competing chains more aggressively.

# 24_3_6
### IMC22 | SADPonzi: Detecting and Characterizing Ponzi Schemes in Ethereum Smart Contracts
* Motivation: The authors concentrate on analyzing and identifying Ponzi scams of smart contracts on Ethereum.
* Background: The Ponzi scam represents financial scams that raise money through a growing downline.
* Ideas: The authors first define four types of implementation patterns, involving, handover scheme, chain scheme, tree scheme, and withdraw scheme. Then they use the program analysis technology to identify ponzi contracts based on the pattern.
* Design: The authors first abstract the dataflow and type information from their pre-defined pattern. Then they adopt symbolic execution analysis technology to match the pattern with the target contract.
* Implementation: The authors achieve their design by developing a system named SADPonzi. SADPonzi integrates Manticore to analyze the contract based on the four types of patterns.
* Evaluation: The authors further apply SADPonzi to all 3.4 million smart contracts, and identify 835 Ponzi scheme contracts, with over 17 million US Dollars invested by victims.

# 24_3_2
### SEC19 | Tracing Transactions Across Cryptocurrency Ledgers
* Motivation: The author focuses on tracing the cross-chain transaction of cryptocurrency incurred by ShapeShift to evaluate  Anonymity and the motivations of crime.
* Background: ShapeShift is a centralized exchange supporting transactions of cross-chain cryptocurrency.
* Data collection: The author collects their data from the webpage of ShapeShift and its API.
* Approach: First, the author uses their heuristics algorithms and predefined patterns to recover the transaction relationships between different blockchains. Then, the author adopts their heuristics cluster algorithm such relationships to identify the account entity
* Applications: The author applies their approach in four aspects. 1) Starscrape's rug pull scam. 2) Stoling coins scams. 3) Trading bots detections. 4) measurement of anonymity. e.g., this approach detects 107 trade bots with seven pairs of coins.


# 24_2_22
### SP24 | SMARTINV: Multimodal Learning for Smart Contract Invariant Inference
* Motivation: The author tries to detect the flaws of smart contracts based on invariable.
* Background: Invariable specifies the variable must remain unchanged on the execution of the contract. Such is the variable in an assert condition.
* Idea: The author adopts LLM to detect such flaws.
* Design: The author decouples the finetuning process to find invariable. The author adopts the process of the expert discovering flaws to finetune and prompt LLM by a fine-tuning approach.
* Approach: The author develops SMARTINV to achieve their idea.
* Evaluation:  SMARTINV uncovers 119 zero-day bugs and six of them are high severity.

# 24_2_21
### FSE23 | EtherDiffer: Differential Testing on RPC Services of Ethereum Nodes
* Motivation: The author focuses on inspecting the inconsistent implementations of RPC services between four implementations of Ethereum.
* Background: RPC services are the front nodes of the Ethereum, and the user invokes DApp by RPC services.
* Idea: The author adopts differential testing technology to find inconsistent bugs in RPC services in geth, besu, nethermind, and Erigon.
* Challenge: How to generate test cases?
* Solution： The author constructs a domain-specific language (DSL) to capture syntax and semantic information. The DSL can generate valid and invalid test cases.
* Approach: The author develops EtherDiffer to achieve their idea. EtherDiffer constructs a local network with those four clients and generates the transactions to mock the non-deterministic on-chain environment.
* Evaluation: EtherDiffer detected 48 different classes of deviations including 11 implementation bugs
such as crash and denial-of-service bugs.



# 24_2_12 
### SP24 | Nyx: Detecting Exploitable Front-Running Vulnerabilities in Smart Contracts
* Motivation: The author concentrates on detecting exploit front-running vulnerabilities of smart contracts.
* Background: Front-running vulnerabilities represent that malicious users leverage prior knowledge of upcoming transactions to execute attack transactions in advance and make profits.
* Prior work: The prior work only detects one single function that caused false positives.
* Idea: The author detects a pair of functions to check whether the contract has the flaw.
* Challenge: The large search space hinders the detection
* Solution: The author formulas some patterns to prune the space.
* Approach: The author develops the Nyx to achieve their idea. Nyx first transforms the contract to Slither IR, and generates a system dependency graph (SDG). Then Nyx filters the function pairs and leverages the z3 SMT to detect front-running flaws.
* Evaluation: Nyx unveils the four 0-day flaws.


# 24_2_11
### ISSTA23 | DeFiTainter: Detecting Price Manipulation Vulnerabilities in DeFi Protocols
* Motivation: The author focuses on detecting the price manipulation flaws in smart contracts.
* Idea: The author uses the dataflow analysis to restore the intra-contract and inter-contract call graph.
* Design: The author first restores the call graph by dataflow analysis, then adopts the flaws pattern and taint analysis to check the flaws.
* Implement: The author develops the DeFiTainter to achieve their idea. They use GigahorseIR and Datalog to conduct taint analysis
* Evaluation: DeFiTainter uncovers three previously undisclosed price manipulation flaws.


# 24_2_9
## ISSTA23 | Definition and Detection of Defects in NFT Smart Contracts
* Motivation: The author focuses on detecting the flaws in NFT smart contracts.
* Idea: The author conducts surveys to summarize five flaws in NFT smart contract, and uses program analysis technology to inspect these flaws.
* Design: The author first uses symbolic execution technology to key variables and program logic, then encodes the flaw's rule by SMT to detect the flaw.
* Implement: The author develops the NFTGuard to achieve their idea. 
* Evaluation: The experimental results show that NFTGuard finds 1,331 smart contracts in the dataset to contain at least one of the flaws. Furthermore, the overall precision achieved by NFTGuard is 92.6%.

# 24_2_6 
## CCS19 | Learning to Fuzz from Symbolic Execution with Application to Smart Contracts
* Motivation: The author focuses on improving the efficacy of smart contracts fuzzing.
* Idea: The author leverages machine learning to obtain the feeds of fuzzy from symbolic execution, which helps fuzzy can achieve more coverage of the smart contract.
* Design: The author first symbolically executes a smart contract to get some invoking sequences (i.e., transactions or inputs). Then they use a neural network to learn the invoking sequences, and fuzzing smart contract with such sequences.
* Implement: The author develops the ILF to achieve their ideas. They use go-ethereum to replay sequences, and construct a network by themselves.
* Evaluation: The ILF is evaluated on 18k smart contracts, which outperforms the existing fuzzer in code coverage and speed.

# 24_2_5
## ICSE24 | GPTScan: Detecting Logic Vulnerabilities in Smart Contracts by Combining GPT with Program Analysis.
* Motivation: The researcher studies how to detect logic flaws in the smart contract by leveraging the ChatGPT.
* Idea: The author proposes a fine-granular prompt approach to submit the flaw description to ChatGPT. Concretely, the authors submit function-level specifications and the reason for vulnerabilities separately, which the authors believe allows Chatgpt to understand the semantics of the code. Next, the static analyzer will evaluate the results from ChatGPT.
* Challenge: The token amounts of the smart contracts are too large, which will cause a large financial cost in training on ChatGPT.
* Solution: The authors use a heuristic filtering scheme to reduce the number of functions of smart contracts that need to be checked.
* Approach: The author develops the GPTScan to achieve their ideas. The GPTScan first filters sensitive functions, then uses GPT to detect the semantics of the contract and infer key variables and finally passes it to a static analyzer to do the confirmation of the vulnerability.
* Evaluation: GPTScan achieves high precision and finds 9 new vulnerabilities.
  

# 24_2_4
## ISSTA22 | eTainter: Detecting Gas-Related Vulnerabilities in Smart Contracts
* Motivation: The author researches the types of smart contracts, which cause the high cost of gas.
* Prior work: The prior work (i.e., MadMax) researches the high cost of smart contracts with fixed patterns, e.g., unbound loop. However, such works can be bypassed via subtle code changes.
* Idea: The author deems the high gas costs are caused by the user's input and access to persistent storage.
* Design: The author leverages taint analysis and static analysis technology to determine whether the user's input reaches persistent storage multiple times. Besides, the author with
* Implement: The author implements eTainter based on the proposed design and MadMax.
* Evaluation: eTainter to perform analysis of 60,612 smart contracts on the Ethereum blockchain. It finds that gas-related vulnerabilities exist in 2,763 of these contracts.

# 24_2_2
## NDSS24 | Abusing the Ethereum Smart Contract Verification Services for Fun and Profit
* Motivation: The author researches the inconsistency between the service of contract verification (e.g., Etherscan) and the real source code of the contract.
* Background: The service of contract verification will display the source code of the contract submitted by the developer, but the adversary can submit the malicious source code of the contract to deceive the verification service and mislead the user.
* Design: The author summarizes the six steps of the verification service, and abstracts the three security properties of such verification service including data integrity, data consistency, and unambiguous Reference.
* discover: Based on the three security properties the author unveils eight vulnerabilities of verification services, which the adversary can exploit to deceive the verification service. The eight vulnerabilities of verification services are caused by compilers, EVM design, and libraries, etc.
* Evaluation: The author conducts the experiment of Verification Services into Etherscan, Sourcify, and Blockscout. They uncovered 19 exploitable vulnerabilities in total on such services and lead to a financial loss of $7.2M.
 

# 24_1_25
## NDSS24 | Not your Type! Detecting Storage Collision Vulnerabilities in Ethereum Smart Contracts
* Motivation: This work aims to analyze automatically and exploit a type of flaw called storage collision vulnerabilities in smart contracts.
* Background: The storage collision vulnerabilities are caused when the proxy invokes of smart contracts don't have type check.
* Design: The author adopts symbolic execution technology to inspect and exploit such flaws.
* Implement: The author implements CRUSH framework based on the proposed design. The author first uses symbolic execution technology to infer the semantics of the contract and uses backward slice technology to analyze the exploitability of such flaws.
* Evaluation: CRUSH detects such flaws on 14,237,696 smart contracts and unevil financial lost about 6M USD by storage collision vulnerability.

# 23_12_6
## CCS23 | Lanturn: Measuring Economic Security of Smart Contracts Through Adaptive Learning
* Motivation: This research explores how to maximize the benefits of MEV, that is, how to order transactions to maximize benefits.
* Design: The author takes the transaction sorting problem into a black-box optimization problem, and then uses adaptive machine learning technology to optimize it.
* Implement: The author implemented the Lanturn framework based on the proposed design. Lanturn is deployed on the validated node. It first optimizes the transaction sequence, then simulates the execution of the transaction sequence, then optimizes the transaction sequence based on the execution results, and then executes the transaction again until the profit reaches the maximum.
* Evaluation: Lanturn two new MEV strategies.

# 23_11_25
## CCS23 | TxPhishScope: Towards Detecting and Understanding Transaction-based Phishing on Ethereum
* Motivation: The author empirically studies a new phishing attack on Ethereum, TxPhish.
* Background: The author defines Txphish as a scam in which users visit a phishing website and are then induced to send transactions, resulting in the loss of their own crypto assets.
* Approach: The author designed a system, TxPhishScope, to detect TxPhish scams. First, TxPhishScope collects domain names through certificate transparency and filters domain names based on keywords in web content. Then TxPhishScope triggers transactions on suspicious websites, and finally determines whether the website is a TxPhish scam based on the results.
* Observation: TxPhishScope successfully detected and reported 26,333 TxPhish websites (78.9% of them were first reported) and 3,486 phishing accounts.
* Insight1: The author reveals that TxPhish websites have three characteristics, encompassing short lifespan, low cost, and fast update frequency.
* Insight2: phishing fund flow demonstrates that 54.0% of phishing funds ($43.7 million) flowed into centralized exchanges.
* Mitigation: TxPhishScope identified bugs in six Ethereum projects and provided criminal evidence of four phishing accounts (involving $1.5 million) to aid in the recovery of funds. 


# 23_11_22
## Sec20 | EthBMC: A Bounded Model Checker for Smart Contracts
* Motivation: How to enhance the capabilities of existing smart contract formal verification tools.
* Prior work and Challenges: Existing work has the following problems: the hash function cannot be modeled, the memory model modeling is not accurate enough, and cross-contract analysis cannot be performed.
* Solutions: For the hash function, the author converted it into a set of memory-based constraints; for the memory model, the author proposed that the author build a graph-based memory model (similar to a data flow graph). For cross-contract calls, the author believes that based on Accurate memory modeling can solve this problem.
* Design: The author abstracts and models the execution environment of the contract, including accounts, memory, persistent storage, contract calls, and hash functions, and then symbolically executes them.
* Implementation: The author implemented the EthBMC framework based on the proposed design. ETHBMC is implemented in around 13,000 lines of Rust code and uses SMT Yices2 as a solver.
* Evaluation: The author used ETHBMC to detect three types of vulnerabilities: Ether theft, control flow hijacking, and self-destruction, and found that 1983, 54, and 1474 contracts had the above problems respectively.

# 23_11_20
## CCS16 | Making Smart Contracts Smarter
* Motivation: The author focuses on detecting security issues in smart contracts, that is, how to use symbolic execution technology to determine that smart contracts do not contain four types of security issues.
* Background: The author introduces four categories of threatened contract vulnerabilities, including Transaction-Ordering Dependence, Timestamp Dependence, Mishandled Exceptions, and Reentrancy Vulnerability.
* Design: The author first formalized the basic semantics of EVM, including instructions, blocks, transaction execution, etc., then proposed four types of security issue detection rules based on this formal model, and finally constructed symbolic execution tools based on this formal system to detect corresponding security issues.
* Implement: The author implemented the Oyente framework based on the proposed design. Oyente first builds a CFG graph based on the contract bytecode, and then simulates the execution of the contract based on the symbolic model, in which the solver uses z3.
* Evaluation: The author used Oyente to detect 19 366 smart contracts and found that 8833 contracts potentially have the proposed bugs.

# 23_11_18
## CCS20 | ACE: Asynchronous and Concurrent Execution of Complex Smart Contracts
* Motivation： The author focuses on how to execute more complex smart contracts faster.
* Challenge: Challenge: Smart contract execution needs to be verified on all blockchain nodes.
* Prior research: Arbitrum_{usenixsec18} randomly selects some nodes from the nodes set as verification nodes to improve the performance and Ekiden_{enrosp19} performs contract execution in TEE to ensure confidentiality.
* Key idea: The author's main idea is to let the contract owner randomly allocate the verifiers for contract execution, and it does not require all verifiers to reach a consensus, but only satisfies the Byzantine consensus.
* Design: Step 1, the user executes the transaction locally and obtains the contracts involved in the transaction; Step2, the miner determines the order of transaction execution based on the contract dependencies; Step3, the miner appoints transaction execution to nodes trusted by the contract owner for execution; Step4, after the execution result of the trust node reaches the Byzantine consensus, the result is submitted to the miner for uploading to the chain.
* Implement： The author implemented the ACE framework based on the proposed design for concurrent and offline execution of transactions.
* Evaluation: The concurrency capabilities of the ACE framework can be 100 to 10,000 times faster than Ethereum's sequential execution, and require less storage, communication, and computing resources.
  
  

# 23_11_15
## IMC23 | Ethereum's Proposer-Builder Separation: Promises and Realities
* Motivation: The author analyzes whether the Proposer-Builder Separation (PBS) mechanism plays a decentralized and censorship-resistant role.
* Background: The PBS mechanism decouples the ability of miners (validators) to order transactions in blocks to two new role and replay, the builder. The PBS design goals are two-fold: (1) prevent censorship, and (2) decentralize transaction validation by not giving large entities an advantage in block building.
* Method: The author collected transaction pool data, MEV data, and relay data for analysis.
* Observation1: PBS allows professional amateur builders to compete with large entities, but the builders and relays themselves remain centralized.
* Observation2: PBS does not play a censorship-resistant role because the transaction amount of the sanctioned address involved in the non-PBS block exceeds the transaction amount of the PBS block.
* Insight: The author deems that most of the relays in the industry are not trustworthy, as proposers cannot receive 100% of the profits and the replays are not audit-resistant.

# 23_11_11
## ICSE24 | Revealing Hidden Threats: An Empirical Study of Library Misuse in Smart Contracts.
* Motivation: The author focuses on security issues related to library misuse in smart contracts.
* Method： The author first collected reports from contract auditing companies, summarized common library misuse patterns in eight types of contract development, and finally looked for cases of library misuse patterns in contracts on the Ethereum.
* Insght1: The library misuse patterns are widespread in the real world and the author analyzed 1018 real-world contracts and found 905 misuse cases in 456 contracts.
* Insght2: 99% of all identified cases are concentrated in three subtle patterns. The most common pattern occurred 543 times in 204 contracts, with 83 of those contracts containing the same type of misuse in different code locations.
* Suggestion: Some libraries that lack maintenance or do not trust the source are still used by many contracts.

# 23_11_9
## SOSP21 | Forerunner: Constraint-based Speculative Transaction Execution for Ethereum.
* Motivation: The author focuses on how to use speculative execution technology to improve the performance of Ethereum.
* Challenge: The transactions packaged in each block are unpredictable, which requires speculative execution technology to predict a large number of transaction execution results.
* Solution： The author found that the contracts called by a large number of transactions all executed the same code, so by extracting a small number of different data flows and execution flow results, it was possible to cover all transaction execution results to the greatest extent possible.
* Design: First, in the consensus phase, all possible transaction execution results are analyzed through speculative execution technology, and the sequence of these possible executions is converted into a new, faster execution intermediate representation, and then these intermediate representations are used in the transaction execution phase to achieve faster transactions executing and state transferring.
* Implement: The authors implemented their design based on the go-ethereum client, dubbed the Foreruner.
* Evaluation： Forerunner can bring six times acceleration to Ethereum transaction execution.

# 23_11_5
## SP20 | Executable Operational Semantics of Solidity
* Motivation: The author focuses on the correctness of the semantic implementation of the solidity language.
* Method: The author formally implemented and reconstructed the semantics and syntax of solidity based on the rewriting logic of the K framework.
* Evaluation: The author evaluates the completeness of their formalized solidity semantics through solc compiler test set.
* Application: The authors applied the solidity semantic implementation they proposed to security testing such as compiler bugs and reentrancy vulnerability detection.

# 23_11_2
## IMC18 | Measuring Ethereum Network Peers
* Motivation: The authors measured the topological characteristics and network characteristics of the Ethereum execution layer P2P network.
* Background: The protocol of the Ethereum network is divided into three layers, including RLPx for node discovery and secure transport, DEVp2p for application session establishment, and the Ethereum application-level protocol (a.k.a, subprotocol).
* Method： The author developed nodedefinder based on the Ethereum client geth. Nodefinder reconstructs the network implementation limitations of geth, making it easier for the author to collect more information and data on the Ethereum P2P network.
* Measure： The author deployed NodeFinder from April 18–July 8, 2018 and collected two datasets: 1) a comprehensive view of the Ethereum ecosystem via the nodes seen within the full 82 day collection period and 2) a 24 hour snapshot view of the peers on the main Ethereum network to understand the network’s instantaneous properties.
* Observation1：The author ultimately discovered 228% more nodes than in prior work.
* Observation2: Fewer than half of DEVp2p nodes contribute to the main Ethereum blockchain and there are a large number of nodes in the main network running unofficial versions and unstable versions of clients.
* Observation3: The author found that geth and parity have inconsistencies in the calculation implementation of kad distance.

# 23_10_26
## Sigmetric23 | Strategic Latency Reduction in Blockchain Peer-to-Peer Networks
* Motivation: The author focuses on the latency problem of the Ethereum P2P network and the economic security caused by the delay problem.
* Key idea: The authors define two basic types of latency and propose their solutions for mitigating latency.
* Model： The author defines two latency types: direct latency and triangle latency. Direct latency refers to the delay between nodes, and triangle latency refers to the latency when one node is a relay of the other two nodes.
* Mitigation： The author optimizes the delay between nodes based on the Peri scheme. Specifically, each node is required to actively relay blocks, otherwise the node will be forced offline.
* Evaluation: The author found that Peri technology can optimize half of the latency of P2P networks. At the same time, the author proves that it is difficult to design a strategy-proof peering protocols.

# 23_10_25
## WWW23 | BERT4ETH: A Pre-trained Transformer for Ethereum Fraud Detection
* Motivation: The author focuses on how to universally detect financial fraud in Ethereum.
* Key idea: The author uses machine learning technology to extract timing features of Ethereum to detect financial fraud, instead of conventional graph features.
* Design： The author uses Transfermer to extract the characteristics of the transaction sequence, then uses BERT for pre-training, and finally detects the corresponding fraud problem.
* Challenges: The duplication and skew alleviation of blockchain dataset.
* Solutions: The author uses a higher mask rate and drop rate in the pre-training stage to solve the problem of repeated data, and the author designs a comparative loss function to solve the skew alleviation problem.
* Implement: The authors proposed BERT4ETH based on their proposed method. 
* Evaluation: BERT4ETH achieved a high hit rate on phishing account detection and de-anonymization tasks.

# 23_10_24
## SP20 |  Compositional Security for Reentrant Applications
* Motivation: The author focuses on composable security of smart contracts such as reentrancy.
* Key idea: The author uses control flow integrity (CFI) technology to ensure the integrity of data across contracts and modules.
* Design: The control flow integrity mechanism designed by the author requires that all state changes be completed when trusted code calls untrusted code, and no further state (invariant) changes can be made when calling back trusted code from untrusted code. At the same time, the author proposes static and dynamic locking schemes to protect key invariants.
* Implement: The authors proposed SeRIF based on their proposed method. The authors used JFlex and CUP to parse the contract source code, and then used SHErrLoc as the theorem solver to analyze the integrity of the control flow.
* Evaluation: The author evaluates SeRIF on four cases, and all of them were able to identify the corresponding reentrancy vulnerabilities.

# 23_10_18
## CCS23 | Phoenix: Detect and Locate Resilience Issues in Blockchain via Context-Sensitive Chaos
* Motivation: The authors focus on how to detect resilience issues in blockchain systems. Resilience refers to the ability of the blockchain system to return to normal state after encountering an abnormality.
* Key idea: The author introduces chaos engineering technology to test the resilience of the blockchain system.
* Design: The author uses chaos engineering to test the resilience issues of the blockchain at the consensus level, network level, and blockchain level. When a problem is found, the corresponding problem sequence is collected for replay and analysis of the problem.
* Implement: The authors proposed Phoenix based on their proposed method. Phoenix can deploy on Hyperledger Fabric, FISCO-BCOS, Quorum, Go-Ethereum, and Binance Smart Chain.
* Evaluation: Phoenix has detected 13 resilience issues and successfully generated reproduction sequences for all of them.

# 23_10_9
## SP20 | VerX: Safety Verification of Smart Contracts
* Motivation: The author proposes a new framework to ensure the security properties of smart contracts using formal verification techniques.
* Key idea: The authors construct a new symbolic execution engine to more accurately and efficiently model the semantics of the Ethereum Virtual Machine.
* Challenge： How to solve the infinity of contract test transactions during the verification process.
* Solution： The authors collect constraints during symbolic execution and calculate the fixed points they eventually reach to avoid endless exploration and testing.
* Design: First, users need to provide security requirements (specification) and the contract to be tested. The specification needs to be split and inserted into the contract, and then symbolic execution of the new contract is performed. If the contract can hold the specifications during the symbol period, the verification passes. If not, the author will collect path constraints and determine whether these path constraints can meet security requirements (reachability problem).
* Implement: The authors proposed VerX based on their proposed method. 
* Evaluation: VERX successfully verified 12 real-world projects (138 contracts) with 83 properties author considered.

# 23_10_7
## SP21 | SmartPulse: Automated Checking of Temporal Properties in Smart Contracts
* Motivation: Ensure the security properties and liveness of smart contracts.
* Key idea: The author designed a user-friendly specification language that allows users to specify the rules that the contract needs to meet; and it is equipped with a corresponding property check system to verify the satisfiability of the rules.
* Design: The author developed a specification language SmartLTL. Users use SmartLTL to specify specifications, then insert the specifications into the contract, then convert the contract into standard LTL, and finally use the counterexample-guided abstraction refinement (CEGAR) paradigm for property check.
* Implement: The authors proposed SMARTPULSE based on their proposed method. SMARTPULSE first uses the VeriSol Solidity-to-Boogie translator to convert the solidity code into a Boogie intermediate representation, and then uses the UltimateAutomizer software model checker to detect the Boogie level.
* Evaluation: The authors use SMARTPULSE to check 1947 properties across 191 smart contracts and show that SMARTPULSE is effective at verifying/falsifying these properties

# 23_10_6
## CAV22 | SolCMC: Solidity Compiler’s Model Checker
* Motivation: How to improve the security of smart contracts during the compilation phase.
* Approach: Integrating the formal verification tool SolCMC into the solc compiler allows the formal verification tool to play a role in the compilation phase, conduct more security checks on smart contracts, and provide more security warnings to developers.
* Implement: The author converts the AST into a CFG composed of Horn clauses, and then uses theorem solver and model checking tools to perform automated vulnerability reasoning. Based on assertions and require conditions, the tool detects issues such as out of bounds, and the lack of underflows, overflows, divisions by zero, and transfers with insufficient balance. Detect issues such as reentrancy or self-destruction based on testing tools.
* Evaluation：The author use Beacon Chain Deposit Contract and the OpenZeppelin implementation of the ERC777 to demenstrate the effectiveness of the tool.

# 23_10_2
## ISSTA23 | iSyn: Semi-automated Smart Contract Synthesis from Legal Financial Agreements
* Motivation: The author studies how to convert textual legal agreements into smart contract programs.
* Challenge: How to automatically preserve the semantics in legal agreements into the implementation of smart contracts.
* Solution: The author implements a new IR (intermediate representation) for semantic abstraction of legal agreements.
* Design: The author summarizes four common types of semantic rules in four types of legal agreements, abstracts these semantic rules into an IR, and then converts all legal agreements into this IR. The author implements different template functions for these different IRs. Depending on the IR, different smart contracts will be generated.
* Implement: The authors implemented the method they proposed into the framework iSyn. The authors used a large number of different NLP technologies to extract the semantics of legal agreements, then established SMARTIR to describe these semantics, and finally implemented different Solidity templates for different ITEMs of IR.
* Evaluation： The author tested on 86 legal agreements, and the average length of the generated contracts was 179 lines, and the semantic consistency was as high as 90%.

# 23_10_1
## pldi23 | Automated Detection of Under-Constrained Circuits in Zero-Knowledge Proofs
* Motivation: The authors focus on detecting underconstrained vulnerabilities in zero-knowledge circuit implementations. Underconstrained vulnerabilities refer to the situation where multiple solutions exist in zero-knowledge circuits, resulting in malicious users being able to construct malicious proofs, making the proof verification fail.
* Key idea： The author found that the input of most zero-knowledge circuits only corresponds to the output, and the functions of these circuits can be decomposed into functions composed of inputs and a series of intermediate variables.
* Design: The author proposes a method to detect under-constrained vulnerabilities. This method first expresses the relevant variables in the circuit equation as functions and inputs. When there are relevant variables that cannot be expressed as functions, the solver is used to construct equivalent functions; continuously The above process is repeated iteratively until a vulnerability is discovered.
* Implementation： The authors implemented the QED^2 tool based on the method they proposed.
* Evaluation: The evaluation shows that QED^2 can successfully solve 70% of 163 arithmetic circuits from Circomlib and detects 8 vulnerable templates that can be underconstrained.


# 23_9_30
## Sec24 | Practical Security Analysis of Zero-Knowledge Proof Circuits
* Motivation: The author focuses on vulnerability detection in zero-knowledge proof circuits, specifically circuits programmed by the language Circom.
* Key idea： The author first collected vulnerabilities in existing zero-knowledge circuits to classify and model them, and then converted the zero-knowledge circuit into a graph, on which vulnerability patterns were matched and detected.
* Design: The author divides the vulnerabilities of existing zkp circuits into three categories, including Unsafe component usage, Constraint-computation discrepancies, and Nondeterministic signals. Then the author abstract the program of the zero-knowledge protocol circuit into a graph structure, namely Circuit Dependence Graph (CDG), and established a set of predicate semantic detection rules for CDG. Based on predicate semantic detection rules, the author creates different anti-patterns for the three types of vulnerabilities mentioned above and then performs vulnerability detection on the graph.
* Implementation： The authors implemented the ZKAP tool based on the method they proposed.
* Evaluation: The author implements 9 different detectors using this framework and performed an experimental evaluation on over 258 circuits from popular Circom projects on Github. 


# 23_9_29
## Eurosys23 | Diablo: A Benchmark Suite for Blockchains
* Motivation: The author focuses on how to truly measure the performance of blockchain platforms.
* Prior work： Performance measurement in the industry always over-exaggerates blockchain performance and does not disclose experimental configurations, while performance measurement in academia tends to focus on single-platform testing and is not generic.
* Key idea： The author designed a set of test benchmarks and test environments that include a variety of real DApps of different types and languages, and proposed a unified strategy framework.
* Design： The author uses smart contracts to simulate applications such as Dota, Uber, and FiFA to construct real blockchain loads, and then measures the performance of multiple types of blockchains based on different experimental environments (e.g., atacenter, testnet, devnet, community and consortium).
* Implement: The authors implemented a framework called Diablo based on their proposed approach. Diablo consists of three parts. The master is used to generate load (transaction) summary results, slaves are used to execute load (transaction) and collect structures, and the underlying blockchain platform is used as the test object.
* Evaluation: The blockchains are not capable of handling the demand of the selected centralized applications when deployed on modern commodity computers across the world.
* Insight: The authors found that the performance of current blockchain platforms is highly dependent on the underlying experimental settings in which they are evaluated. The language abstraction level supported by some blockchain platforms is low and unfriendly, and only EVM is very versatile.


# 23_9_28
## CCS23 | Fuzz on the Beach: Fuzzing Solana Smart Contracts 
* Motivation: The author focuses on how to use fuzz testing to detect security issues in Solana contracts.
* Prior work： The author was the first to conduct research on fuzzing Solana smart contracts.
* Key idea： The author attempts to implement a solana contract fuzz testing tool based on code coverage guidance for bytecode.
* Design： The author first constructed the on-chain environment to simulate account and contract information, etc., then mutated the transaction sequence and fuzzed the contract in Solana's eBPF virtual machine.
* Implement: The authors developed a tool, FuzzDelSol, to implement their proposed method. FuzzDelSol uses libalf to guide fuzz testing and implements the detection of six vulnerabilities including, i)missing signer check, ii) missing owner check, iii) arbitrary cross program invocation, iv) missing key check, v) integer bugs, vi) and a lamports-based bug oracle to detect lamport-theft.
* Evaluation: The evaluation of 6049 smart contracts shows that FuzzDelSol’s bug oracles find impactful vulnerabilities with high precision and recall.

# 23_9_27
## OOPSLA23 | Synthesis-powered optimization of smart contracts via data type refactoring 
* Motivation: The author mainly focuses on how to save contract gas by modifying the definition of contract variables (data layout).
* Approach: First, the user manually specifies how the data types of the current contract need to be decoupled; then a semantically equivalent sketch contract (i.e., the sketch contract refers to the contract that is marked with changes to the data structure and its impact statements) is generated based on the contract changes, and then the author uses the sketch contract to generate a semantically equivalent and more gas-saving contract.
* challenge： How to use sketch contracts to generate semantically equivalent and more optimized gas contracts contracts?
* Solution： The author converts the problem of searching for semantically equivalent and more optimized gas contracts into a MaxSAT problem to solve.
* Implement: The author implemented the approach in a tool called Solidare, which is implemented in a combination of Java and Kotlin. Solidare uses the Sat4j tool to solve the generated MaxSAT instances.
* Evaluation: Solidare is able to reduce the gas usage of some Solidity programs by up to 48.6%.

# 23_9_26
## OOPSLA23 | Asparagus: Automated Synthesis of Parametric Gas Upper-bounds for Smart Contracts
* Motivation: The author focuses on how to accurately estimate the gas upper bound of contract calls to avoid a series of problems caused by out of gas.
* Approach: The author first establishes a state transition equation on the CFG of the contract function to obtain gas constraints, and then uses mathematical theorem to simplify the gas constraints to obtain a quadratic gas estimation parameter equation.
* Implement: Asparagus first builds a CFG in the form of RBR (Rule-based Representations), then converts the CFG into a PTS (Polynomial Transition System), and then generates the corresponding gas constraints based on the PTS and utilizes the mathematical theorems (Farkas’ Lemma, Handelman’s Theorem, and Putinar’s Positivstellensatz) obtains the final quadratic parameter gas estimation polynomial.
* Evaluation: The asparagus approach can handle 80.56% of the functions (126,269 out of 156,735) in comparison with GASTAP's 58.62%. 

# 23_9_25
## CCS23 | Demystifying DeFi MEV Activities in Flashbots Bundle.
* Motivation: The author mainly focuses on the security of the Ethereum economy, that is, studying how to analyze and restore MEV (Miner Extractable Value) behavior, and studying the impact of MEV activities on the blockchain.
* Background: Maximum Extractable Value (MEV) is the maximum value that can be extracted from block production in excess of standard block rewards and gas fees by adding and excluding transactions in the block and changing the order of transactions in the block.
* Previous work: Existing work uses fixed patterns and can only identify known MEV activities.
* Design: The author first uses fixed financial behavior rules to identify known financial behaviors, and then uses machine learning to cluster unknown new MEV behaviors.
* Implement: Based on the method proposed by the author, the author implemented the tools ActLifter and ActCluster. ActLifter identifies ten known financial activities (such as token swaps, liquidity additions) and their asset transfer rules through contract events and fixed financial behavior rules, and ActCluster uses a clustering algorithm to explore previous financial behaviors and asset transfer rules. Unknown MEV behavior.
* Evaluation: ActLifter’s DeFi behavior recognition capabilities and ActCluster clustering new MEV behaviors can achieve nearly 100% precision and recall. At the same time, the authors obtained many new observations using the above two tools and discovered 17 new MEV activities that have not been reported yet.

# 23_9_23
## ISSTA22 | WASAI: uncovering vulnerabilities in Wasm smart contracts.
* Motivation: The author focuses on the security issues of WASM (WebAssembly) smart contracts on the EOS blockchain platform.
* Design: The author uses symbolic execution to generate the input on the trace of WASM contract, and then uses the input as a seed for fuzz testing on WASM contract.
* Challenge: Specificity of the underlying architecture of WASM smart contracts, such as memory models, data types, etc.
* Solution: The author uses hooker and collect the specific values (trace) of the relevant structures when the WASM contract is executed to avoid symbolizing.
* Evaluation: The authors developed the tool WASAI to implement their method, WASMI is twice as fast as existing tools and discovered a CVE (CVE-2022-27134).

# 23_9_21
## sec21 | SMARTEST: Effectively Hunting Vulnerable Transaction Sequences in Smart Contracts through Language Model-Guided Symbolic Execution.
* Motivation： The author mainly focuses on detecting smart contract vulnerabilities and generating corresponding transaction sequences to verify the corresponding vulnerabilities.
* Key idea： Generate transaction sequences through statistical language models to explore program paths more effectively.
* Design: The author first learns the transaction sequences that trigger the vulnerability through a statistical language model, and then uses these transaction sequences to guide the path judgment process of symbolic execution.
* Implement: The authors implemented their design into the tool SmartTest, trained the n-gram language model to generate a model for transaction sequences, and then used the VeriSmart tool to perform symbolic execution. SMARTEST supports the detection of six types of security-critical vulnerabilities: integer over/underflow, assertion violation, division-by-zero,
ERC20 standard violation, Ether-leaking vulnerability, and suicidal vulnerability. 
* Evaluation: SmartTest's vulnerability detection rate is as high as 90.5%, which is much higher than tools such as Mythril.

# 23_9_20
## FSE23 | Demystifying the Composition and Code Reuse in Solidity Smart Contracts.
* Motivation: The author empirically studies the code reuse problem during smart contract development, specifically the code type, purpose and deployment method reused by developers.
* Method: The author collected 350,000 contract projects. Because a project may consist of multiple contracts, the author divided them into externally contracts and self-developed contracts for research.
* Find1: 80% of contracts come from external references, of which NPM accounts for the majority.
* Find2: Less than 5% of the code of self-developed contracts is customized.
* Find3: 35% of the external contracts are related to standard interfaces (such as ERC20), but there are a lot of interface misuses.
* Find4: The author extracted 61 common reuse contract deployment scenarios, such as using uniswap code to obtain permissions, etc.

# 23_9_19
## ICDCS23 | Smart Contract Parallel Execution with Fine-Grained State Accesses.
* Motivation: The author proposes a series of technologies to optimize the concurrent execution of contracts and optimize the performance of the blockchain.
* Prior work: The optimistic concurrency control (OCC) repeatedly executes transactions until there are no transaction conflicts, which is inefficient.
* Challenge: Transactions with state read and write conflicts will hinder the efficiency of concurrent execution.
* Key idea: The author detects the conflicts of each transaction by constructing the contract's state read/write dependency graph, and then finds the optimal transaction execution sequence.
* Design: Through SDG, the read and write dependencies between transactions can be analyzed. The author mainly uses SDG to resolve to write and write conflicts. Specifically, the author records the write operation of each transaction, and then automatically finds the required transaction for conflicting transactions. In addition, conflicting transactions can perform required write operations from SDG in advance without having to wait for the transaction to be submitted to get the result.
* Implement: The author implemented their design as DMVCC. For a blockchain node, DMVCC first builds SDG based on contracts and transactions, then uses SDG information to sort transactions, and finally executes the sorted transactions.
* Evaluation: The experimental results show that DMVCC doubles the parallel speedup achievable to a 20× overall speedup, compared with the serial execution baseline, approaching the theoretical optimum.

# 23_9_18
## sec23 | Your Exploit is Mine: Instantly Synthesizing Counterattack Smart Contract.
* Motivation: The author studies real-time defense against attacks on smart contracts on the blockchain, specifically, attacks initiated through vulnerabilities in contract implementation.
* Design: The author assumes that relevant attack transactions have been detected, so its main idea is how to identify attack contracts, generate countermeasure contracts, and then front run the attacker's malicious transactions.
* Challenge: How to identify the contract and remove the attacker's access control when generating a counter-contract.
* Approach: For the identification of attack contracts, the author finds the attack contracts by having short deployment time and data flow dependencies between attack transactions. Regarding the attacker's access control when generating the contract, the author bypassed it by replaying the attack transaction and finding the jump statements and transaction attribute parameters (msg.sender or tx.origin).
* Evaluation: The evaluation with 62 real-world recent exploits demonstrates approach's effectiveness, successfully countering 54 of the exploits. 

# 23_9_17
## sec23 | A Mixed-Methods Study of Security Practices of Smart Contract Developers.
* Motivation: The author investigates and analyzes current smart contract developers’ perceptions of security.
* Method: The author first conducted semi-structured interviews and code audit tests on 29 participants, and then conducted an online questionnaire survey on 171 participants.
* Observation1: Current developers do not pay attention to the security of smart contracts. The reasons include that they pay more attention to efficiency, the code is forked from well-known projects, and they rely on security audits.
* Obervation2: Developer awareness of security vulnerabilities is high, with less than half of the participants identifying security vulnerabilities in the audit tasks designed by the authors.
* Obervation3：Simply relying on existing measures such as documentation, reference implementations, and security tools is not enough to help junior developers identify contract security vulnerabilities.
* Insight1: The industry needs to invest more resources in helping developers learn and understand security vulnerabilities.

# 23_9_16
## ISSTA23 | Toward Automated Detecting Unanticipated Price Feed in Smart Contract.
* Motivation: This article focuses on the security issues of blockchain oracles, specifically how to defend against price manipulation problems caused by oracles in real time.
* Key Idea: The author uses formal modeling of the oracle, and then uses actual transactions to test the security of the oracle.
* Design: First, developers need to predefine the specifications of the price oracle and the corresponding verification rules themselves, and then use the transactions in the memory pool to perform symbolic execution in real time using the k framework, and detect whether there is price manipulation based on the amount of tokens in the fund pool.
* Implement: The author implemented their method into a tool VeriOracle.
* Evaluation: VeriOracle is effcient in that its verifcation time (about 4s) is less than the block time of Ethereum (about 14s).

# 23_9_15
## ISSTA23 | Detecting State Inconsistency Bugs in DApps via On-Chain Transaction Replay and Fuzzing
* Motivation: The author focuses on state inconsistency vulnerabilities in smart contracts. Specifically, state inconsistency refers to state where the contract does not follow the expectations of developers or users. For example, front-running, reentrancy, and lack of access control are all such vulnerabilities.
* Prior work: Previous research using static analysis has extremely high false positives, while using dynamic analysis requires a sound oracle.
* Key Idea： The authors use historical transactions and their context (state, contract calling relationship) for fuzz testing users to discover such vulnerabilities. In order to avoid using oracles, the author uses the consistency of the state after each transaction sequence is executed to determine whether there are loopholes in the contract.
* Design and implement: The authors implemented their method into a tool IcyChecker. IcyChecker first collects transaction traces, then generates and mutates transaction sequences to fuzz test the Dapp contract, and determines whether there are vulnerabilities in the contract by looking at the state consistency before and after mutation.
* Evaluation： IcyChecker identifies a total number of 277 SI bugs with a precision of 87% on the top 100 popular DApps. nine of the flaws have new attack patterns which are never disclosed by
prior research and are reported to corresponding authorities.

# 23_9_14
## ICSE23 | AChecker: Statically Detecting Smart Contract Access Control Vulnerabilities
* Motivation: The author mainly focuses on the access control issue of smart contracts, that is, accessing key variables and codes, and whether the contract implements reasonable access control.
* Key Idea: The author associates the user role of the contract with related state variables to restore the implementation of state access.
* Design： The author first uses backward data flow analysis and heuristic rules to obtain the state variables and specific implementation of access control, and then uses symbolic execution and taint analysis to see whether the state variables can be bypassed to access sensitive code or variables.
* Implement: The authors implemented their method based on eTainter and named it AChecker.
* Evaluation: The author verified the effectiveness of AChecker on three data sets: CVE, Smartbug, and Popular contract. AChecker can flag vulnerabilities in 21 frequently-used contracts on the Ethereum blockchain with 90% precision.

# 23_9_11
## ISSTA23 | SmartState: Detecting State-Reverting Vulnerabilities in Smart Contracts via Fine-Grained State-Dependency Analysis.
* Motivation: The author focuses on the identification of a type of vulnerability in smart contracts, State-reverting Vulnerability (SRV).
* Key Idea： The author constructed a new state dependency graph (SDG) based on state reading and writing and control flow relationships, and used historical transaction data to help complete the construction of the graph, and then performed SRVs detection based on the graph.
* Design: The author first relies on the read-write relationship of state variables to build SDG in the function, then builds the SDG edges between functions based on control flow and data flow, and then builds some edges that cannot be found by static analysis based on historical transaction guidance. Finally, the author determines whether sensitive state variables lack access control and have uncertain state dependencies on the graph to determine whether there is SRV.
* Implemention: The author built a tool SmartTest to implement the method it proposed.
* Evaluatin：SmartTest achieves a precision of 87.23% and a recall of 89.13% and detects 11 SRVs on the real contracts.
  
# 23_9_10
## WWW23 | Know Your Transactions: Real-time and Generic Transaction Semantic Representation on Blockchain & Web3 Ecosystem.
* Motivation: The author focuses on how to identify the high-level semantics of blockchain transactions, such as token exchange and liquidity addition.
* Key Idea： The author uses the graph analysis method (network motifs) to first define some basic patterns, and then use these basic patterns to mine token transaction flows.
* Design and Implementation: The author first reconstructs the RPC API of the blockchain client into a parallel design in order to improve efficiency; then extracts transaction flow information from the blockchain, including contract creation, token transfer, eth transfer and other information, to form a semantic graph. Finally, 16 basic patterns were defined, and then advanced semantic mining and inference were performed on the semantic graph.
* Evaluation：The method proposed by the author can extract nine categories of high-level semantics, which can then be applied in transaction classification and account classification.

# 23_9_9
## FSE23 | DeepInfer: Deep Type Inference from Smart Contract Bytecode.
* Motivation: The author is mainly concerned about the problem of functional interface recovery of closed source contracts. Specifically, the author hopes to recover the type and number of parameters and return values from the contract bytecode.
* Key idea: The author uses static analysis for feature extraction, and then uses machine learning techniques to perform type analysis.
* Challenge: Nested complex types have no fixed rules and are difficult to infer, and the return value requires the entire function to be inferred.
* Method: The author uses multiple neural networks to play different roles at different stages to solve the above Challenges.
* Design and Implemention: In the first step, the author uses Gigahorse to upgrade bytecode to SSA's IR, then extracts type features (constants, RD relationships) on the IR, and finally builds a CFG on the IR; in the second part, the author first uses the Word2Vec model to restore simple types and complex types Classification of nested types; in the third step, the author uses the sequence generation model to dynamically restore complex nested types; in the fourth step, the author uses the GAT network to learn the return value type from CFG.
* Evaluation: The system designed by the author is superior to existing systems in terms of accuracy and precision, and can adapt to different compiler versions.

# 23_9_8
## issta23 | Beyond “Protected” and “Private”: An Empirical Security Analysis of Custom Function Modifiers in Smart Contracts.
* Motivation: The author investigates and detects the security of modifiers in solidity smart contracts. The modifiers refer to some key conditions at the entrance of the contract function, similar to assert() in C language.
* Challenges: The data dependencies of complex contract state variables and the calling relationships between functions.
* Key Idea: The author constructed a modifiers dependency graph (MDG), abstracted the modifiers into a function, and then used the dependency relationship before the variables and the calling relationship of the function to connect the modifiers with the global state variables and functions into edges, and then in Solve security issues on MDG.
* Design and Implementation: Step 1, the author first uses Slighter to convert the source code into IR, builds MDG on Slighter's IR, and then iteratively constructs new edges (some edges require multiple constructions to obtain) until a fixed point is reached. Step 2, the author uses Z3 to solve on MDG and detects whether any modifiers can be bypassed. Step 3: The author uses NLP technology to cluster the names of decorators and then classify the current decorators.
* Evaluation: The system analyzes a large dataset of 62,464 contracts, over 400 of which were identified with bypassable modifiers.

# 23_9_7
## sec23 | Token Spammers, Rug Pulls, and Sniper Bots: An Analysis of the Ecosystem of Tokens in Ethereum and in the Binance Smart Chain (BNB).
* Motivation: The author mainly focuses on the ecology of tokens in the liquidity pools of Ethereum and Binance Chain, including token life cycle, token spammers, and token trading bots.
* Method: The author ran the full nodes of Ethereum and Binance Chain to collect transaction data, and then collected token information based on the token interface and token events; collected token transaction pair information based on Dex events.
* Insight1：About 60% of tokens are active for less than one day, and 1% of addresses create an anomalous number of tokens (between 20% and 25%).
* Insight2: The author discovered a new scam called a 1-day rug pull, which is a rug pull that occurs in one day. This scam generates $240 million in profits
* Insight3: The author found that some token trading robots in the market are becoming victims of rug pull because they buy tokens without restriction.

# 23_9_6
## ASE23 | DeFiWarder: Protecting DeFi Apps from Token Leaking Vulnerabilities.
* Motivation: The author studies the problem of Token Leaking vulnerability, specifically, the problem that users can withdraw more tokens from DApp abnormally.
* Key Idea: The author analyzes the token transaction flow based on on-chain transactions and calculates the ratio of the user's deposit amount and withdrawal amount to detect Token Leaking vulnerability.
* Implement： The author first extracts the contract call tree in the transaction. The edges are the call relationships of the contracts, and the nodes are the logs of the token contracts. The tree is then heuristically pruned to exclude irrelevant data. Finally, the author merges the token transaction flow from the tree and determines the existence of Token Leaking Vulnerabilities based on the ratio (i.e., withdrawal amount / deposit amount).
* Experiments: The system proposed by the author successfully reveals 25 Token Leaking vulnerabilities from 30 Defi apps.

# 23_9_5
## sec23 | Automated Inference on Financial Security of Ethereum Smart Contracts.
* Motivation: The author focuses on how to universally detect vulnerabilities in token contracts.
* Existing work： The program static analysis technology is limited to fixed pattern matching and is not universal; while automatic reasoning technology only intelligently detects a single vulnerability and does not have scalability.
* Key Idea: The author abstracts the relevant variables of the token, uses automatic reasoning technology to detect the rationality of these abstract variables, and has achieved the purpose of multi-vulnerability detection.
* Implement： The author first uses the K framework to abstract the source code of the token contract, then divides the variables of the token contract into invariant properties (e.g., the total amount of balance) and equivalence property (e.g., the balance of each account), then uses Tamarin prover for automatic reasoning, and finally Use z3 to check whether the relevant variables meet the corresponding constraints.
* Experiments: The system proposed by author can detect transaction order dependency (TOD), timestamp dependency(TD), Reentrancy, gasless send, overflow/underflow, and transferMint, and the accuracy to identify token contracts is higher than 98%.

# 23_9_4
## sec23 | Smart Learning to Find Dumb Contracts.
* Motivation: The author studies the problem of smart contract vulnerability detection, specifically, how to extend the source code level analysis capabilities of existing formal detection tools to bytecode.
* Key Idea: The author uses static analysis tools to perform supervised learning at the source code level, then compiles the source code into bytecode, and then performs machine learning by matching the source code-level vulnerability results with the bytecode.
* Implement： The system proposed by the author first uses slither to analyze the contract source code, then compiles the source code into bytecode, and then associates the bytecode with slither's detection results.
* Challenge: For close source contracts, the author will perform similarity analysis twice, the first time It is used to filter out those bytecodes with vulnerabilities that are similar to those that have been detected, and the second test is for vulnerability detection.
* Experiments: The system proposed by the author leads the pack with a 100% Completion Rate, 99.7% Accuracy, and 0.2 second average contract analysis time, i.e., 10-1,000x faster than competitors. Besides, the True Positive Rate is 98.7% and False Positive Rate is 0.6%.

# 23_9_3
## sec23 | Snapping Snap Sync: Practical Attacks on Go Ethereum Synchronising Nodes.
* Motivation: This research focuses on the security issue of the synchronization mechanism of the Ethereum client (i.e., go-ethereum), that is, by interfering with the synchronization mechanism of Ethereum to achieve the purpose of forking the blockchain.
* Key Idea: The author uses some defects in the synchronization mechanism of the Ethereum client (geth), such as the chain import mechanism, the state pruning mechanism, and the segmented block verification mechanism, to construct three types of different attack schemes.
* Scope: geth in Ethereum mainnet before PoS fork, Ethereum Classic, Ethereum PoW.
* Implementation: The first Attack method uses the block import mechanism and state pruning mechanism to invalidate the legal chain; the second attack method uses the characteristics of the client to randomly verify blocks, constructs false blocks, and attacks consensus; third attack method is a combination of the first two.
* Cost: The optimal cost of this attack only requires one millionth of the mining computing power to achieve the purpose of forking the blockchain.


# 23_9_2
## ccs22 | Towards Automated Safety Vetting of Smart Contracts in Decentralized Applications
* Motivation: The author mainly studies the inconsistency between the front-end UI of the DApps and the back-end contract of the DApps, and then does some risk research on the DApps.
* Prior Study: Previous work mainly used heuristic rules to detect Dapp vulnerabilities, which is not universal.
* Approach: The author first uses static analysis to extract the program semantic graph from the DApp's contract code, then extracts the business logic of the contract from the DApp's UI, and then conducts consistency detection and risk verification.
* Innovation： The author uses the UI info to obtain the business logic of the Dapp (i.e., the specifications in formal analysis).
* Evaluation: The author discovers 19 new safety risks in the wild, such as expired lottery tickets and double voting.

# 23_9_1
## raid22 | Penny Wise and Pound Foolish: Quantifying the Risk of Unlimited Approval of ERC20 Tokens on Ethereum
* Motivation: The author studies the status quo, risks, and security issues of unlimited approvals in ERC20 tokens, and proposes a best practice for user-initiated approvals.
* Background： The approval mechanism is used to delegate the privilege of spending users’ tokens to DApps. However, current Dapps usually allow users to grant unlimited approval quotas without the user's knowledge, which exposes users' assets to risk.
* Approach: The author first analyzes the transaction history of Ethereum and defines some indicators (such as amount) to quantify the severity of the current unlimited approvals. Then, for some well-known wallets and exchanges, check their approval process and evaluate the current industry's emphasis on unlimited approval. Finally, the authors propose a set of best practice approval processes for users.
* Insight1：The research reveals that 60% of collected approval transactions belong to unlimited approval transactions, and a majority of unique users (2.9m/4.9m) have participated in unlimited approval transactions.
* Insight2：Only 10% of DApps and wallets provide explanatory information for the approval mechanism, and only 16% of UIs enable users to adjust approval amounts.
* Insight3：The evaluation result suggests that only 0.2% (2,475/1,496,886) of investigated users mitigate risk by following best practices.


# 23_8_31
## ndss23 | Smarter Contracts: Detecting Vulnerabilities in Smart Contracts with Deep Transfer Learning
* Motivation: How to efficiently, fine-grained, and scalable detect vulnerabilities in smart contracts.
* Prior work: Non-machine learning methods, such as program analysis, are inefficient and not scalable; machine learning-based methods rely on source code and intelligently perform binary classification.
* Approach: The author utilizes transfer learning to quickly and lightweightly train a network containing new vulnerabilities while ensuring multi-classification of vulnerabilities.
* Implement： The author first obtains the contract bytecode from the blockchain, then uses the existing vulnerability detection tools (e.g., Oyente, Mythril, Vandal, Maian) to label the contract, and finally sends it to the neural network for training.
* Evaluation： The tool from this study can detect 11 vulnerability classes in 0.15s per smart contract on average and yields an average F1 score of 97% across all evaluated classes.

# 23_8_30
## ndss23 | Double and Nothing: Understanding and Detecting Cryptocurrency Giveaway Scams.
* Motivation: The authors measure and investigate the current state of attackers using websites to phish victims with high cryptocurrency rebates.
* Approach: The author used the certificate information to obtain a large number of phishing websites of attackers, and then conducted a lot of analysis on the information of these websites. In addition, the author also analyzed the relevant transactions of the attacker's wallet account to assess the loss.
* Observation1: During the six months the authors studied, a total of 10,079 phishing sites were observed, an average of 55.7 phishing sites per day.
* Observation2: During the six months the authors studied, attackers have stolen the equivalent of tens of millions of dollars.
* Observation3: The author extracts the transactions corresponding to 2,266 wallets belonging to scammers.

# 23_8_29
## sec23 | A Large Scale Study of the Ethereum Arbitrage Ecosystem
* Motivation: How will the current large-scale arbitrage affect the blockchain ecology.
* Background: Arbitrage is defined as the simultaneous purchase and sale of the same asset in two different markets for advantageously different prices.
* Key idea1：The author proposes a method based on graph analysis to identify arbitrage behavior. First, the author identifies token behavior, then constructs a token transaction graph, then finds whether there is a cycle in the graph, and finally extracts arbitrage information.
* Key idea2：The author replays historical transactions to identify arbitrage opportunities that have existed in the past.
* Observation 1: The income generated by current arbitrage may be enough to make miners evil because it far exceeds the income generated by consensus mining.
* Observation 2: The current arbitrage benefits may make oracle price manipulation attacks easier to implement.


# 23_8_28
## sp22 | Quantifying Blockchain Extractable Value: How dark is the forest?
* Motivation： Quantify and expand the profits of current blockchain extractable value (BEV) and analyze the harm of BEV to the current blockchain consensus.
* Measurement： The author quantifies the profits brought by miner arbitrage, sandwich attacks, and  liquidation.
* Key Idea: The author proposes an algorithm for replayed transactions to estimate the profits that are not calculated by BEV.
* Contribution: The author formalizes the BEV relay concept as an extension of the P2P transaction fee auction model.
* Conclusion: BEV relay not only did not alleviate the network load, but jeopardized the consensus of the blockchain. The algorithm from author extends the total captured BEV by 35.18M USD.



# issta22_SPCon 
## issta22 | Finding permission bugs in smart contracts with role mining.
* Motivation： This research focuses on verifying whether the access control policy design of smart contracts is reasonable.
* Key Idea： The author mines the corresponding role permission structure from historical transactions, and conducts a conformance testing on these role permissions to see if there are flaws.
* Technical challenge: The role and permission information provided by historical transactions is incomplete.
* Solution： The author abstracts the process of recovering from a partial role permission model to a complete model as an optimization problem, and adopts genetic algorithm to obtain the optimal solution.
* Implements and evaluation: The author implements SPCon and evaluates it on the sampled 50 smart contracts within the role mining benchmark. The results show that SPCon can largely reduce the number of mined with better accuracy.

# sp23_WeRLman
## sp23 | WeRLman: To Tackle Whale (Transactions), Go Deep (RL).
* Motivation: This research mainly studies the security impact of the abnormal miner reward brought by MEV on the Proof-of-Work (POW) mechanism.
* Previous Work: Previous work did not consider miner rewards for MEV anomalies, and modeled them with constant rewards.
* Key Idea： The author adopts deep Reinforcement Learning (RL) to build a POW reward model to evaluate possible security issues in the POW mechanism, especially the collapse of the POW reward mechanism caused by MEV.
* Technical challenge： The high reward (i.e., the amount) for miners by MEV will cause the state space that needs to be simulated to become very large.
* Solution: The author obtains a simplified model by simplifying the behavior of miners. The model is two orders of magnitude smaller than the full model and can be solved by dynamic programming.
* Evaluation and conclusion：The authors analyzed the historical records of Bitcoin and reveal that the security threshold of Bitcoin (the proportion of computing power required to attack the network) is declining sharply through their proposed model.

# sp23_CF
## sp23 | Clockwork Finance: Automated Analysis of Economic Security in Smart Contracts.
* Motivation: Detect and identify financial security issues of DeFi smart contracts.
* Previous work: Existing works can only detect known and limited DeFi smart contract vulnerabilities.
* Key Idea: The author uses formal verification techniques to automatically reason about and quantify the financial security problems that contracts may encounter.
* Implements： The author instantiates Clockwork Finance Framework (CFF) in the K framework for mechanized proofs.
* Insight: CFF uncovers on average an expected $56 million of EV per month in the recent past.

# ccs22_vrust
## ccs22 | VRust: Automated Vulnerability Detection for Solana Smart Contracts.
* Motivation： Detect security issues with smart contracts on the solana chain.
* Background: Solana is a stateless blockchain, which requires callers to provide the data they need in the parameters of the interface of the contract, which leads to a new attack surface.
* Key Idea： The author developed a static contract verification tool VRust, after converting the contract into the form of MIR, and then comparing the eight fixed vulnerability patterns proposed by the author to detect security issues.
* Evaluation： VRust has been evaluated on over a hundred Solana projects, and it has revealed 12 previously unknown vulnerabilities, including 3 critical vulnerabilities in the official Solana Programming Library confirmed by core developers

# 23_8_27
## sp23 | Three Birds with One Stone: Efficient Partitioning Attacks on Interdependent Cryptocurrency Networks.
* Motivation： This study explores the threat to the security of the blockchain itself if the autonomous systems (ASes) of the blockchain network is not independent.
* Background： ASes own a series of IPs, usually telecom operators or cloud service providers.
* Key Idea: The author exploits characteristics of different blockchains sharing the same ASes, and further partitions Bitcoin and Ethereum by launching a partition attack on the Ripple blockchain first.
* Technical challenge： Blockchain networks are often overlapping and thus it is not easy to mine topological data.
* Solution: Ripple's network information is publicly disclosed, so attacks are launched through Ripple.
* Evaluation: 9.4K blockchain nodes can be attacked by partitions.

  
