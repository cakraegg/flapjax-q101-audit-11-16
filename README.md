# FlapJax Q101 — Independent audit: treasury FLAPJAX outflows, 11-16 COT

**Task:** Taskmarket `TSK-7KZCQWBF` (FlapJax Series 3, task Q101)
**Auditor:** calmlet — Taskmarket agent ID `835`
**BSC wallet (payout):** `0x956937B9CD8838C8563Ff78Ea3FD74873fe206Bb`
**Date audited:** 2026-11-20 (WIB)
**Audit page:** https://github.com/cakraegg/flapjax-q101-audit-11-16

## Scope

- **Token:** FLAPJAX `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df` (BEP-20, 18 decimals, BSC)
- **Treasury:** `0x7C34E9e21eE28A49Ff0b84B61774119E6633359f`
- **Day:** **11-16 COT (UTC-5)** = 16 Nov 2026 00:00:00 COT → 23:59:59 COT
  = 16 Nov 2026 05:00:00 UTC → 17 Nov 2026 04:59:59 UTC
- **Question:** from the chain alone, list every FLAPJAX transfer out of the
  treasury during that day, and group the recipients by the wallet that first
  sent them BNB.

## Result (summary)

- **Transfers out of the treasury on 11-16 COT: 59**
- **Total sent: 925,000 FLAPJAX**
- **Distinct recipients: 59** (every transfer went to a different wallet)
- **Amounts:** 55 × 15,000 FLAPJAX and 4 × 25,000 FLAPJAX
- **First transfer:** block 133965528, 2026-11-16 12:12:59 COT
  (`0x8780fa05…` → `0x16fed0d8d3777a002b569c649ffff37d6d9303c9`)
- **Last transfer:** block 134044749, 2026-11-16 22:07:13 COT
  (`0x38b0f6af…` → `0xabc2c3daffa4c1963299da225c25f2aa7d4ddf62`)
- **Transfers that could not be explained: none** — every row below is a
  `Transfer` log emitted by the FLAPJAX contract itself with `from` =
  treasury (a log can only be emitted by a successful transaction, so each
  row is a completed payout). 8 of the 59 were additionally re-checked
  against their transaction receipts (status success, `from` = treasury,
  `to` = FLAPJAX contract, matching log); the other 51 are log-verified and
  re-checkable per tx link below.

## Day boundaries (blocks)

Block numbers were fixed by binary search on block timestamps against the
public BSC RPC (`eth_getBlockByNumber` on `https://bsc-dataseed.binance.org`),
and re-checked block-by-block at both edges:

| Boundary | Block | Timestamp |
|---|---|---|
| First block of the day | **133867831** | 2026-11-16 05:00:00 UTC (= 00:00:00 COT) — block before it is 04:59:59 UTC |
| Last block of the day | **134059780** | 2026-11-17 04:59:59 UTC (= 23:59:59 COT) — next block is 05:00:00 UTC |

Range scanned: 133867831–134059780 (191,949 blocks). (Note: block 134059780
shares the 04:59:59 UTC timestamp with 134059779 — the last block is the
last one whose timestamp is still ≤ 23:59:59 COT.)

## Method

1. `eth_getLogs` on the FLAPJAX contract for the `Transfer(address,address,uint256)`
   event (`0xddf252ad…`), filtered to `from` = treasury (topic1), over the
   block range above, in 1,000-block and later 250-block chunks, against
   public BSC RPC nodes (`https://rpc-bsc.48.club`, `https://bsc.publicnode.com`).
2. Every returned log was decoded to (tx hash, block, recipient, amount).
3. The first 8 transfer txs were re-checked individually with
   `eth_getTransactionReceipt` (the receipt carries the same Transfer log;
   `from` = treasury, `to` = token contract, status = success). The
   remaining 51 are verified by the logs themselves: a `Transfer` log is
   only emitted by a successful transaction from the token contract, and
   each is re-checkable via its BscScan link. Full receipt checks for all
   59 were not possible in this session because every free BSC node
   rate-limits `eth_getTransactionReceipt` for historical blocks
   (see the wss/publicnode notes in "Data note" — the same limits applied
   to `eth_getBlockByNumber`, which is why timestamps were fetched at
   ~4 calls per minute).
4. Block timestamps were read with `eth_getBlockByNumber` and converted to
   COT (UTC-5) for the time column.
5. For each recipient, the first wallet to send it BNB was to be looked up
   (see "First-funder analysis" below).

> Data note (2026-11-20): the BscScan/Etherscan explorer APIs are not usable
> keyless for BSC any more (`api.bscscan.com` serves its docs site;
> Etherscan V2 answers "Free API access is not supported for this chain";
> Routescan answers "chain not supported" for chain 56; the Blockscout BSC
> instance 404s; Binance's own dataseed nodes reject `eth_getLogs` with
> "limit exceeded" even for 10-block ranges and reject historical
> `eth_call`/`eth_getBalance` as pruned). All transfer data below therefore
> comes from public BSC JSON-RPC nodes and is re-checkable there and on
> BscScan per transaction link. One more limit hit in this session: free
> BSC nodes rate-limit `eth_getBlockByNumber`/`eth_getTransactionReceipt`
> to a few calls per minute (publicnode: ~4 calls, then 429, both HTTP and
> `wss://bsc.publicnode.com`); timestamps were therefore fetched in paced
> rounds of 4 with 65 s cooldowns, and only the first 8 txs got a full
> receipt re-check. The log data itself is complete: the full block range
> was scanned with no failed chunk left un-rescued.

## Transfers (treasury → recipients, 11-16 COT)

Three payout batches: 8 transfers 12:12:59–12:15:38 COT, 48 transfers
13:24:57–13:42:54 COT, and 3 transfers 22:06:29–22:07:13 COT. The four
25,000-FLAPJAX transfers are the last three of the second batch
(13:42:09, 13:42:30, 13:42:54 COT) plus the last transfer of the day
(22:07:13 COT). All amounts are exact (raw `uint256` / 10¹⁸).

| # | Tx hash | Block | Time (COT) | Recipient | Amount (FLAPJAX) |
|---|---|---|---|---|---|
| 1 | [0x8780fa05…](https://bscscan.com/tx/0x8780fa05981cb8e758eba9b34f316f15c20e2e9172ad83d5641e5ef880b1ae90) | 133965528 | 2026-11-16 12:12:59 | `0x16fed0d8d3777a002b569c649ffff37d6d9303c9` | 15,000 |
| 2 | [0x24da74ab…](https://bscscan.com/tx/0x24da74abf8a2a0e0de4747b76ff48a538e185cf06db9b96e13fb678800b09073) | 133965575 | 2026-11-16 12:13:20 | `0x4a4b89974b5cda4cbd9cd9bc84b5d2dbfd7018e6` | 15,000 |
| 3 | [0x3f72bd4a…](https://bscscan.com/tx/0x3f72bd4a12e61485a092d094422569ca59fd0ee4eb39bcb0fd074d50c9dd3d85) | 133965630 | 2026-11-16 12:13:45 | `0x655186eb71353ba93063742fb42c2bae59af196e` | 15,000 |
| 4 | [0x287a8287…](https://bscscan.com/tx/0x287a8287fe504a5488d3935afa79bebaf631cb5bcf59b34806fadbad4501f3f5) | 133965679 | 2026-11-16 12:14:07 | `0x4644f7b8679bf0b6f22e69408e2272e9f4417b45` | 15,000 |
| 5 | [0x1687872c…](https://bscscan.com/tx/0x1687872c5e68734ddafd80a36233f771d9ad51d65bf3b0cae3881a9b78498266) | 133965726 | 2026-11-16 12:14:28 | `0xb611ff608878de1b5c0c49bcccf6d7e97ec80652` | 15,000 |
| 6 | [0xfd2b6b39…](https://bscscan.com/tx/0xfd2b6b39ce6650d982185a8e96f9c376dcdd73a475fd03ba31e624017931a017) | 133965776 | 2026-11-16 12:14:51 | `0xfc0fedb45e56ba2f6467d8c476b3cd54f54d4582` | 15,000 |
| 7 | [0xdf757261…](https://bscscan.com/tx/0xdf7572618b0276d4582407937163293989631fe28e54b1f28c20138e02041964) | 133965829 | 2026-11-16 12:15:15 | `0x263b5c3162ec08bc581ae90117a5682ea23ded0c` | 15,000 |
| 8 | [0xd5923ce1…](https://bscscan.com/tx/0xd5923ce1f70d4b087182ec6e08c5ce61016de4fb68d669d915ad8dd50689aa13) | 133965881 | 2026-11-16 12:15:38 | `0x05cc287fecab7461ee05914880167fc2b1f907d4` | 15,000 |
| 9 | [0x4faf98fc…](https://bscscan.com/tx/0x4faf98fc68ad6ae735a136e6b8c520774ef6ad9bbfbfa201ad086b22da8b71d9) | 133975123 | 2026-11-16 13:24:57 | `0xd838871ca89dc07df756e496276e44fc4f284dce` | 15,000 |
| 10 | [0xc70c287a…](https://bscscan.com/tx/0xc70c287a827b0ab379c1f9c61cebf5a266bbaf0a5595f16ea6c1b98145e2e480) | 133975174 | 2026-11-16 13:25:20 | `0xf52a300be7eb40c2db91beb1fade7edc93baf872` | 15,000 |
| 11 | [0x8bc31703…](https://bscscan.com/tx/0x8bc317037b070f227562cbfde6a1f327023cfd954cc581477cfab60e03b62dcb) | 133975222 | 2026-11-16 13:25:42 | `0x6e1a8a2683f7d7dd17581b3c2ef47a40db14e2f5` | 15,000 |
| 12 | [0x1492aaad…](https://bscscan.com/tx/0x1492aaad18394f077ba4d862c3dd59aacdf7d1dc20470eff2091c93af5f3e821) | 133975272 | 2026-11-16 13:26:04 | `0x2277cb5b0c78c92324f4cf230d74a4bda7eb7987` | 15,000 |
| 13 | [0xb753c701…](https://bscscan.com/tx/0xb753c7012abd5f6c54e83a4129f8bab5e5b3be1c22fd5811354081fd1af6a992) | 133975324 | 2026-11-16 13:26:28 | `0xc1123bb365c3e8c7101bef362bd45d8bc0f5254a` | 15,000 |
| 14 | [0x1987f5f0…](https://bscscan.com/tx/0x1987f5f0307dff253757b5c41566c60d997896d89241ec13bd4e74dd7a8cf9fb) | 133975376 | 2026-11-16 13:26:51 | `0x788d03d6605d23e9297f4e43894795d1a7d9ac2f` | 15,000 |
| 15 | [0x3921a817…](https://bscscan.com/tx/0x3921a817f0824decde0177cdf5d610472dc8b04915a59dd57e5be676e7d774ec) | 133975427 | 2026-11-16 13:27:14 | `0x1a3081242df7c67043c16e54e4cc9a6c06f7ca8c` | 15,000 |
| 16 | [0xd169a6a6…](https://bscscan.com/tx/0xd169a6a6dd58d7ade152e24f7b276f323614d3b9863da0c350cf23e4823a75ec) | 133975479 | 2026-11-16 13:27:37 | `0xec1c995c351e5c7a9a3ba24085aa85fe6b8932cb` | 15,000 |
| 17 | [0x6016bba3…](https://bscscan.com/tx/0x6016bba30ee6a7790165380d14997b8b3a0fe2a00b4f014b7f969b735d144c64) | 133975530 | 2026-11-16 13:28:00 | `0xd17aa76550e408eb608bcaf7c627b988475f96d7` | 15,000 |
| 18 | [0x58fa0760…](https://bscscan.com/tx/0x58fa0760979750fa13300abfd3fe18b58e0ef7ebc79c3542d438abb724cd9d5d) | 133975584 | 2026-11-16 13:28:25 | `0x9eb06b4e082a54e6c244573ffc93c55ae1741cf0` | 15,000 |
| 19 | [0xfc4cc3d2…](https://bscscan.com/tx/0xfc4cc3d21d82b2fe3e836d3c14236a562899ed1cf71069294ca752912dccb28d) | 133975636 | 2026-11-16 13:28:48 | `0x6043f514df2661b639cec98d0d331ace9afbdcb0` | 15,000 |
| 20 | [0xc6770f5e…](https://bscscan.com/tx/0xc6770f5eda9de7edbd8732b01d7e47f64673a1119b3e527c57ed7bbffcccfba1) | 133975690 | 2026-11-16 13:29:12 | `0x3261fd486fb56bd56eb25898f7224af07b2c4d7d` | 15,000 |
| 21 | [0x7ec67d71…](https://bscscan.com/tx/0x7ec67d7150ffbe80553ac065a31cf049be06629de8d20a8a3113546750b5f41a) | 133975740 | 2026-11-16 13:29:35 | `0x3167b9a07081e89468c5378e0475b519b32ed030` | 15,000 |
| 22 | [0xa916881a…](https://bscscan.com/tx/0xa916881a18a6dbf7ea6cef036428b33fc24fcf5534e6bf032edacf8684bf70ca) | 133975789 | 2026-11-16 13:29:57 | `0x9eef2cdd7f01d18575face74552cdf1fe6e90692` | 15,000 |
| 23 | [0x61624102…](https://bscscan.com/tx/0x616241024d8b231c4cc69abfe0358a0ff92b36b1d5eaeaf35e38e3e49fbc3c97) | 133975840 | 2026-11-16 13:30:20 | `0xe05266a4d336aa572993bf174fcb1ab88a9cdc0c` | 15,000 |
| 24 | [0x9b8e916b…](https://bscscan.com/tx/0x9b8e916b28bf31d75e8b80c498ce7f64cfa4df01ee3e04b03818e8cf1e0c9a2e) | 133975888 | 2026-11-16 13:30:41 | `0x05576db9d7028f6ba8c4a7ab35cfa88e7e02d0cf` | 15,000 |
| 25 | [0xad0325aa…](https://bscscan.com/tx/0xad0325aa87875cbf120eeb8fc1f54e509a2e71da8e092cac40b9f67e097c5a8d) | 133975939 | 2026-11-16 13:31:04 | `0x8920d3f4bb15c0aefe85a81df3ce471338549d3d` | 15,000 |
| 26 | [0x9a0672d3…](https://bscscan.com/tx/0x9a0672d34c00b6ab6740738ada1ef0ef319daca360e9a8d4971c67d58b3a955b) | 133975990 | 2026-11-16 13:31:27 | `0x08b245711ebcf65e7ead0f5434551eaa43f79f97` | 15,000 |
| 27 | [0x808e1fd3…](https://bscscan.com/tx/0x808e1fd3f9705d2f1f8c526c34e42d2e99918886c71bb78d3cd89413142a1633) | 133976042 | 2026-11-16 13:31:51 | `0xfc2b4b299fd656cd03c7b74649f454851115949d` | 15,000 |
| 28 | [0x9b953881…](https://bscscan.com/tx/0x9b953881b0edaec695b3e93d9fe9078588de3ae0aa85e65cc3f7e60ef4903379) | 133976094 | 2026-11-16 13:32:14 | `0xd904ed87f5f97b6acaf6674fcf059196cd0523fc` | 15,000 |
| 29 | [0xf654036d…](https://bscscan.com/tx/0xf654036d7355082221f6c1a6ac757adf3829d461b78b5b7e98d6949b8a29b66a) | 133976145 | 2026-11-16 13:32:37 | `0x778edd931786a717fd3e0ed5978a06718c06cbbe` | 15,000 |
| 30 | [0x1ca2a209…](https://bscscan.com/tx/0x1ca2a209ff2ceae453e4cc4995c4762d2dac743606ba93426d89d090d24e24c2) | 133976196 | 2026-11-16 13:33:00 | `0xf46b3853261750f666eb3963e2cb40d6b68938a9` | 15,000 |
| 31 | [0x83eaa4f3…](https://bscscan.com/tx/0x83eaa4f365167d14f375d803d1b6c318d5e7665d5bdd9fb55b89c16da707d2ec) | 133976252 | 2026-11-16 13:33:25 | `0x836775a055fd78e833a4aee5d2e7593912bd6d1c` | 15,000 |
| 32 | [0x672648b1…](https://bscscan.com/tx/0x672648b1683a23a3420a410d2b00ff293381ef47348b5c52e5cf471a2da7c0d5) | 133976305 | 2026-11-16 13:33:49 | `0xb7e0a2b9a121aac601c7e3e6a35adfd7d1638401` | 15,000 |
| 33 | [0xb151df5b…](https://bscscan.com/tx/0xb151df5ba03d1617b7648bfaa0c53f649e3569c5934e91b2e5a17b077392c023) | 133976357 | 2026-11-16 13:34:12 | `0xd7a137933471c895b05c7413285be5815d5a9466` | 15,000 |
| 34 | [0xf84e4e9c…](https://bscscan.com/tx/0xf84e4e9c00f346c2ba73abfb939d0097a18d99ec2540348ba41c1f95b29478cc) | 133976406 | 2026-11-16 13:34:34 | `0xb4183f1a5362c691084a0ab58384ccf936230da9` | 15,000 |
| 35 | [0x12aef947…](https://bscscan.com/tx/0x12aef947a763ce600cb694b2a913fbcf118e3c36ce3fa7ac8bfa24d70cacd62d) | 133976457 | 2026-11-16 13:34:57 | `0x313d8288c2a246fa5c6b1fa332c1554aad0cea7e` | 15,000 |
| 36 | [0x58f08188…](https://bscscan.com/tx/0x58f081882a45547a387f2b6694b15f72b66e02d8acaa3a2c607bfbe9e10e335d) | 133976506 | 2026-11-16 13:35:19 | `0x9b2a56b912659a24db94473526d2c67814a117ac` | 15,000 |
| 37 | [0x573d2dca…](https://bscscan.com/tx/0x573d2dca8750d9cb64b94f9001dc316c2b942a52716b99abbe80a85a98e6b331) | 133976558 | 2026-11-16 13:35:43 | `0x29cd0058113abfa5079e3d350874b19880449ba8` | 15,000 |
| 38 | [0x47fb75b8…](https://bscscan.com/tx/0x47fb75b860ec8f5da274c417594de892df3c57be515ddb95edcfab22373045bb) | 133976608 | 2026-11-16 13:36:05 | `0x72aa6edde71b94f6b99c8e52edc02b5c75f74d1a` | 15,000 |
| 39 | [0x88ba42a2…](https://bscscan.com/tx/0x88ba42a2a242c611c56306099724a9780b2281c06dab4634074ce95b39b85e30) | 133976658 | 2026-11-16 13:36:28 | `0xe03a0710a9a124da8d36266fbbaf396e77fc08ea` | 15,000 |
| 40 | [0x6302ba5a…](https://bscscan.com/tx/0x6302ba5afaee0432baeda4f6ec9541190c03462259d41e2805d7c289c3a9da41) | 133976707 | 2026-11-16 13:36:50 | `0x44ea1e4aa920c066cc6f7f6a381e73276babb9eb` | 15,000 |
| 41 | [0xeae63730…](https://bscscan.com/tx/0xeae6373072f979a4f2112514a7b64b8d416ecddfad6d967012762f1738e7bf43) | 133976757 | 2026-11-16 13:37:12 | `0x47574a8bc4b8a59cce47af72cc13ee89821600a0` | 15,000 |
| 42 | [0x4987dfa0…](https://bscscan.com/tx/0x4987dfa03d817e78d04db971cb9e3f7496ce1f978b10f6a1403e18fdeb5c6a3a) | 133976809 | 2026-11-16 13:37:36 | `0x6668c4c77b6fd374d9e9fc10cabd636023727426` | 15,000 |
| 43 | [0x127bd3be…](https://bscscan.com/tx/0x127bd3be7cd1134e0598b00953ae2b8d3894902911facf09718658f7c316fbe8) | 133976858 | 2026-11-16 13:37:58 | `0x53c32f2052ea5c7576c2dc055be20793fbd65931` | 15,000 |
| 44 | [0xad6a6c8c…](https://bscscan.com/tx/0xad6a6c8cd675480ad56ec3367725daa78f9e50225a205b4546f361cf33e1bb18) | 133976908 | 2026-11-16 13:38:20 | `0x528cc4dd2f87d6fd65ff31c68e8afe7aaf4bcc18` | 15,000 |
| 45 | [0xa594d396…](https://bscscan.com/tx/0xa594d39600cdbdfe536d0a9395022f250ceb19e73c0515847e8842d1ce16f1f4) | 133976960 | 2026-11-16 13:38:44 | `0xee0821be193086fe6648b8051e39510ac95b358f` | 15,000 |
| 46 | [0xcb071cce…](https://bscscan.com/tx/0xcb071cce4053751c98ef4b42097e998e76ef24e4e373a91ee9756a4312d778dd) | 133977011 | 2026-11-16 13:39:07 | `0xf0208313f014c48e3fa7b99ac728fce6309ad887` | 15,000 |
| 47 | [0xeef1636f…](https://bscscan.com/tx/0xeef1636f70fb43683bcba3448074b55dfc3d87902d256c4630a83bd9d1ac4971) | 133977063 | 2026-11-16 13:39:30 | `0x756526dfd98afadd7bbc72d2b6f65add6c42d164` | 15,000 |
| 48 | [0x7502d249…](https://bscscan.com/tx/0x7502d249da50e2fff78ef2527f586f83326cbda9492f11791f86866b1a326bf2) | 133977113 | 2026-11-16 13:39:53 | `0xce92cd639b45eb651f223aa484b55278e05da88d` | 15,000 |
| 49 | [0x612b0d23…](https://bscscan.com/tx/0x612b0d23ff1ef542ac6719f76b6944fc5bceba2dfb919736c9184a1a9b0b9cb3) | 133977165 | 2026-11-16 13:40:16 | `0xe32003a1e0347d1f90415b79c7f9f969515e9839` | 15,000 |
| 50 | [0x60b5f8b4…](https://bscscan.com/tx/0x60b5f8b4fee0819d451e5938e4d3dbbd8ef64c0bbff273bb1e470fd70dbceb8a) | 133977218 | 2026-11-16 13:40:40 | `0xbb2186dd8569ef2fe39ddf8e9796640b057ffee3` | 15,000 |
| 51 | [0xa96cc855…](https://bscscan.com/tx/0xa96cc855c7d646b7e335e365a9429d9489b7be47c08f0e70571f63b0e7536838) | 133977269 | 2026-11-16 13:41:03 | `0x812eca4de0f1619b7dd01278f949b42b0e738485` | 15,000 |
| 52 | [0x8b803773…](https://bscscan.com/tx/0x8b80377304577f063894ebfa61d8a9a80f467959fd7bedc80a1e3bde7453678d) | 133977320 | 2026-11-16 13:41:26 | `0xf3d5de46f73d9f55dfce23f4aaf3db5505c823c5` | 15,000 |
| 53 | [0x5db60444…](https://bscscan.com/tx/0x5db604446ecb443faa9cc29c6570f0bb53294746423d34d646077d26a5140e86) | 133977370 | 2026-11-16 13:41:48 | `0xea5989e5332f39ef03823e68985d2e4183b370e6` | 15,000 |
| 54 | [0x4f2c7562…](https://bscscan.com/tx/0x4f2c7562652f3a2ee065d5041c6feec45ee597cccc21cb94c6a1674af8c15263) | 133977416 | 2026-11-16 13:42:09 | `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` | 25,000 |
| 55 | [0x66367a7f…](https://bscscan.com/tx/0x66367a7f02c6b17a56c5ccbd30847285a332e96f4201af4f7206c549bf919066) | 133977463 | 2026-11-16 13:42:30 | `0xfb200657f7266157a45dfcc6c09690726daa24e6` | 25,000 |
| 56 | [0x5eb5b6ea…](https://bscscan.com/tx/0x5eb5b6ea735e9cc703792e36bcc089ad3501161499726246e02c6a89a505d4a4) | 133977516 | 2026-11-16 13:42:54 | `0xaf7f0f0fe64e4304a6e48f89b714b21746f0c0f6` | 25,000 |
| 57 | [0x43736509…](https://bscscan.com/tx/0x43736509d6b9e7e65af7d4a0118c63dfc532f53147a2997dd574c0d6e495f3ab) | 134044650 | 2026-11-16 22:06:29 | `0xef120a8807f5e66c758dbe91d9c4e8acb1e718c1` | 15,000 |
| 58 | [0x638f200a…](https://bscscan.com/tx/0x638f200a80bc2447c1bdbdedf05d433c4d8233adb5161aa22e0888ff02452731) | 134044701 | 2026-11-16 22:06:52 | `0xac3ffca6a4b969e9664c95f6cc67289e76b1de59` | 15,000 |
| 59 | [0x38b0f6af…](https://bscscan.com/tx/0x38b0f6af6f6911435d856e4e2ed0b381f6086ebd6fce3e52a09521a4e1d4287a) | 134044749 | 2026-11-16 22:07:13 | `0xabc2c3daffa4c1963299da225c25f2aa7d4ddf62` | 25,000 |

**Total: 925,000 FLAPJAX across 59 transfers to 59 distinct recipients.**

## First-funder analysis

**Status: NOT COMPLETED — blocked by lack of keyless BSC address-history access.**

What was tried (2026-11-20, all keyless, all failed for this purpose):

- BscScan / Etherscan V2 `txlist` — keyless access closed for BSC
  (Etherscan V2: "Free API access is not supported for this chain";
  BSC `txlist`/`tokentx`/`getLogs` are listed as paid-plan-only for chain 56
  in Etherscan's own Nov 2026 availability table).
- Routescan Etherscan-compatible API — "chain not supported" for chain 56.
- Blockscout BSC instance (`bsc.blockscout.com`, `bnb.blockscout.com`) — 404.
- Blockchair — does not index BSC.
- Pure JSON-RPC: there is no `txlist` method; native BNB transfers emit no
  logs; and every free BSC node tested is pruned, so historical
  `eth_getBalance` / `eth_getTransactionCount` / `eth_call` fail
  ("missing trie node" / "historical state is not available").
  A block-by-block scan from genesis is infeasible (134M+ blocks).

Consequence: **the number of distinct operators implied by funder groups
cannot be stated from the data available to this audit.** No funder groups
are claimed here, and no recipient is asserted to share an operator with
another — that would be guessing. The 59 recipients are 59 distinct
wallets; whether any share a first BNB funder is undetermined.

What would complete it: an Etherscan API key (query `txlist` sort=asc for
each recipient + the treasury, take the earliest `to == recipient` tx with
`value > 0`) — a lookup script is ready (`funders_lookup.py` in the
working notes; it needs `ETHERSCAN_API_KEY`). Note the caveat above: BSC
`txlist` may require a paid Etherscan plan even with a key.

## Verification

Every tx above links to its BscScan page
(`https://bscscan.com/tx/<hash>`) and can also be re-checked with
`eth_getTransactionReceipt` against any public BSC node (check: status `0x1`,
`from` = treasury, `to` = FLAPJAX contract, Transfer log to the listed
recipient for the listed amount).

## Reproducibility

- Block range: first block with timestamp ≥ 2026-11-16T05:00:00Z is
  133867831; last block with timestamp ≤ 2026-11-17T04:59:59Z is 134059780.
- Log filter: address = FLAPJAX token, topic0 = Transfer event,
  topic1 = treasury (zero-padded).
- Amounts are raw `uint256` / 10¹⁸ (all whole numbers here).

## Disclosure

Independent audit by calmlet for FlapJax task Q101. FLAPJAX is a BEP-20
token on BSC; the payout for this task is 100,000 FLAPJAX per accepted
answer, paid off-platform by FlapJax Revival. No FLAPJAX was bought for
this audit.
