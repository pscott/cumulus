# When does collator produce blocks?
Start collator with ./target/release/cumulus-test-parachain-collator --base-path /tmp/p1c1 --chain=../polkadot/polkadot_chainspec.json --bootnodes /ipv/127.0.0.1/tcp/30333/QmbQMG8VWKumRyXg47UixQiosUMP2nWRsmWRojqDqJTG9t --ws-port 9955

Collator complains that it can't bind port 9955, yet it seems to bind it just fine (apps comes alive when the node starts and slims down when the node dies.)
No blocks are produced
Balances are shown correctly
Cannot convert parameter `tx`...

Start alice relay chain node with ./target/release/polkadot --chain=local --base-path /tmp/relay-alice --alice
Alice and collator peer
Alice begins producing blocks (why? shouldn't she need another validator online?? Or maybe just any peer?)
Collator is importing Alice's blocks
Apps conencted to Alice shows blocks
Apps connected to collator still shows #0

Register Parachain at correct ID 100 (went in in block 52)
Collator still not producing blocks
Apps shows sudo transaction in block 52
Observation: There is a parachains.setHeads extrinsic in _every_ block
Alice's log messages change from always saying
```
Starting parachain attestation session on top of parent 0x70b5174ff77abb62dd8c077d17d3b605fd6fdac32e0b9986c95913d7ba2df9ed. Local parachain duty is Some(LocalDuty { validation: Relay })
```
to sometimes saying
```
Starting parachain attestation session on top of parent 0xdc9a9ec9054b18ec2a61b703181afbf4d114fc2345a69cfd7ec7acbc3896ba05. Local parachain duty is Some(LocalDuty { validation: Parachain(Id(0)) })
```



Start bob on the relay chain just to see what happens.
Bob and Alice don't peer with each other, but both peer with collator (I don't remember this issue on previous runs)
Both Alice and Bob produce blocks
Bob's parachain duty log message switches from relay to parachain 0 just like Alice's does.

--------------------------------------
## Trying to register two parachains

Start alice
Start Bob
Start p0c1
observe all three nodes have peered. I think this is the first time a collator automatically discovered bob
start p1c1
observe all four nodes peer.

Kill both collators because I forgot to `export-genesis-states` (again)
Generate both genesis states
Restart both collators
observe all nodes have 3 peers again :)

Submit registration extrinsic for parachain 0
Observe p0c1 begins collating
Observe p0c1 logs `Possible safety violation: attempted to re-finalize last finalized block` as described https://github.com/paritytech/cumulus/issues/26
Observe p1c1 is still not authoring, but still syncing relay chain as expected

Submit registration extrinsic for parachain 1
Observe p1c1 begins collating
Observe p1c1 also reports possible safety violation

So far everything has worked as expected this time around.
May as well keep experimenting while we're winning.
Compiling parachain 2
Observe both relay chain validators are saying Local parachain duty is Some(LocalDuty { validation: Relay }) and never saying a specific parachain ID
Remember to export genesis state :)
Start node
Observe it has four peers (as do all other nodes of course)
Observe it is not collating
Submit Registration extrinsic for p2
Observe p2c1 starts collating

Just observed that none of the collators are actually successfully building a blockchain. Each one of them keeps saying
```
Prepared block for proposing at 1 [hash: 0x76f6c3b50d41cdff7fc6cb53d93c8c7ded926b1a6e64e2d13e5487ead4103f52; parent_hash: 0xaee8…7f1a; extrinsics: [0xde71…254e]]
2019-11-29 15:37:14 Possible safety violation: attempted to re-finalize last finalized block 0xaee8680b63df8ee7488a0018c8614ddd5095c67c323e2bc15a894f14e82e7f1a
```
One all three of the collators, they are actually creating a new block every time, but they never move on to block two. This has not been the case in previous runs.

--------------------------------------
## Trying to register two relay chains (again)
Repeating the same experiment as previously

Start clean Alice-Bob relay chain
Observe they peer and begin producing blocks

Start p0c1
Observe 2 peers, syncs relay chain, no collation
Sumbit registration extrinsic
Failed to submit extrinsic. Failed to convert `tx` between node and runtime. I've never had this on the relay chain before. WTF
Also can't submit a balanace transfer transaction

Kill everything, purge all chains.
Restart just alice
CAn't make balance transfer. Grrr. I'm using local apps and the same polkadot build. WTF.
Alice's node says likewise: Failed to submit extrinsic: Extrinsic verification error: Execution: Could not convert parameter `tx` between node and runtime: Invalid transaction version
Kill and restart Apps, now it won't even connect to a node.
Grr. Restarting the whole computer.
