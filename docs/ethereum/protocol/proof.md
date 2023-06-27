---
layout: page
title:  "Provable historical state"
permalink: /ethereum/protocol/stateproof
toc: true
---

In this post let us now look at the innards of proofs. If someone sends you
a proof for some piece of state in Ethereum, what form does it take?

We will walk through naively, starting with a proof and dissecting its parts.

The context is as follows:

Bob: "I want to replay an old block, but the EVM accesses state values that have since changed."

Alice: "Here, take this state that was valid at the time that this old block was produced."

Bob: "Ok, but make sure it matches the state root of the block header, otherwise I can't trust the
values"

  - [Which state?](#which-state)
    - [Verifiability](#verifiability)
    - [Account proof parts](#account-proof-parts)
    - [Degarbling a proof](#degarbling-a-proof)
    - [Decoding the first node](#decoding-the-first-node)
    - [Decoding the second node](#decoding-the-second-node)
    - [Decoding the last node](#decoding-the-last-node)
    - [RLP-Decoded proof](#rlp-decoded-proof)
    - [Getting the value from the proof](#getting-the-value-from-the-proof)
    - [Verifying the proof hashes](#verifying-the-proof-hashes)
    - [Showing the structure of the proof](#showing-the-structure-of-the-proof)
  - [Proper navigation: Using the path](#proper-navigation-using-the-path)
    - [Path navigation at leaves](#path-navigation-at-leaves)
    - [Exclusion proofs](#exclusion-proofs)
    - [Extension nodes](#extension-nodes)
  - [Similarity amongst storage proofs](#similarity-amongst-storage-proofs)
  - [Automation](#automation)
  - [Account proof](#account-proof)
    - [Account proof path](#account-proof-path)
    - [Account proof leaf](#account-proof-leaf)
    - [Conclusion](#conclusion)



## Which state?

Bob can start replaying the transactions of an old block, but he will encounter EVM operations
that access state. For example `SLOAD` means that he will have to load state from storage
for the EVM to access.

Load from where?

Normally a node has a database, but this peer has no node. Do they need a node? If they
are just replaying a single block then a whole node doesn't seem necessary.

Instead, they can potentially request only the values relevant to the transactions in that block.

Alice has replayed the whole block using a node and now knows which parts of state are important.
She bundles those up and can send them to Bob.

### Verifiability

Alice makes sure to include the proof for all the relevant accounts (and the storage required for each account).

Consider block `17_190_873`. She replays the block and identifies that `162` unique accounts and `1679` unique storage slots were accessed. Access meaning read or written to at least once. The
value on first read/write is what Bob is interested in. He can keep track if it was modified multiple times in the block as he replays the transactions in order.

The state root in the block header is the state prior to the execution of the transactions. So for
every account, Alice can link the account state back to this root. Then transactions can be applied
by Bob using these values.

Account `0x431a...528b` is one of the `162` accessed in block `17_190_873`. Alice calls `eth_getProof` and includes every storage key that is useful for that block. There are `9` in total.

Hence the proof that her node responds with looks like this:


```json
    "0x431a2b6ff34a865b6cd7f498e371e2510691528b": {
      "balance": "0x1ae361fc1451c0000",
      "codeHash": "0x26d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543",
      "nonce": "0x1",
      "storageHash": "0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b",
      "accountProof": [
        "0xf90211a0cddf489c063d666cac6218e7059d42beaca7ed5eb5fe8d6fc0b09353c25db199a02ba7f8e5426258da2a51327c018db462ccb059da51eb9bdd84da4cc241298eada0b50c1cc0ce7cd5195ad856d3003dbb8c24852d4f4d6e77151548326cd416fc71a08c080f1bd59931fe8d32d275b678dfe51ac6adb6dcb15ebf516c4c81784d006da0318f22a4e670a7c8e577f71ae7720944d989d661d25d6e39c2ee3f69c50bb82fa07f5901b077738a0629cb3c0350673cf3b3dcea06d6475434575f605beafb4a40a060fddfe2f6d0ab542001878eb89a5d9308033a49522f207f5f0c1fd6a9938544a04d1779e5d5bb0673ff8f76f1dd0952cc98e33e74bad35cdc508ef60d8203af97a0e97f4d0739dc2fc64a84defa064f6200519f6cce7a350711ffc00c4fa9e355bba0e64de506951857c5d5faa34ae91173e2739c0185102367685ede543bb1237277a03551c85d2d49127e70c4eda38184046eac4c9bef5a5a8a08e9722e57aeaa1cd7a01eb2c01d2ff70b62951b30c0a78affcfb6d6416865ff72b4deb7b6a32aca5a3ca096911875c66150e05274005df18c37c33679d960260d82e494941b441ed11082a0f1d58ab8af4d60fa255221c35cbceea2c07eab25cb9e6584f7b1f3d162e370baa0674a08ae583eaa6011049430c8a442ddd8abeb330e0a162636fd6b642fd43535a056061ecbc123c7a4aee4536b853c707173437e270a0d3c9815231b5b7c375c4180",
        "0xf90211a0da98b5edee8d02791916e17aa5c2e7fbe6980a3ca7092cbc550d49bc90e989b9a0c0bb7df96dde21a0a7076f1cd7de42b8e6dfeed06b961e12af8d888c9c8db38da05a1d4c62a6ce02593b51ae60b8cbc7e5a69ef7691eaa8d296d00b20855ba8c3ca07c6ec70610eb4535f6352d0c1035c8bd80f1d6a5e97fb50c154e7fe6161ce5d4a0938b10c17c52bc2449ab06ed4f15a332b5f70cca3e82379edd6fe3cf3f9abe2da09d0e11d6c7619d8683a7fc512814d2b9444c71365c42d680f88d9967b8b5b191a0bcca01aff2daf54c8c1d1ad0fd0cd866e846ea0f6204d279a0145280fa6037daa005862f9d7d6cd30bf386c9e19357ab582815d1930132ece351c3c786b5e7fb59a00a47e0b0764ce95fb9792b3e1fd11be8481b2f0651478a653dee9626c544e883a0c6e24cf72c63eecf6508c8861c1c50ab29cb23e7b2e27df78dabcf540b42ad16a0664310d6777b7d9090db3bd4fe13f514485ad82242befcf4d595520f785b3693a005cfc08b9eeda5589e4f2a95632ea75ed719f1f4ee90bee3fcc9361ccdc9b69da084c17c754a866b4519142878f6109e411dea944f1c511495ae00c0bdd44bcb6fa0caf3202077ee74e5cc76db8803b5981b577ed77ac46c8b081fb07c86584d22c5a039bb4b9886493cbf04e47ab8c63fd07b92ecb59d6ca951043043187ceb9eda9ea06194c79292c32f2e5d486108ecc65d92c70d113df59a2983acc1a23b35d38e2580",
        "0xf90211a0d4ba325a7ca2ec784e600caf840a18a6ee5956c6f592c9ace81b2ee73c6b50c7a01f81cfd22cc189fb623c9b094534b9144ebf3fa866a942e93e5a64af77454cbba0e0dccf44816183de61db475434888e74cc0f455f04e7e562f24467790685e630a04b9d822048492c21c67411161f776e1daa28aec8ae1ece2fe97f4c54b1521fa5a06cb6d01b979a366811474b0d78ea6c41c0424f5a5913420739e270fd27a9f4dba0aa99008c3c67a57a20849e6445d83d2e1d7ab127fb537cbf94ba43d27aa2b2aca0a62b4da8252d825e8b96da9ea5624bdc6d91ce70768746607bbfa79b20608ceba096c3c654d2d0bf1db044563c8399a784ead9016a5332c01a6e5ca16408bc7aa7a09b0f5a65b9024cb3713c79f895ca6bd570413112f1ed35255e8b42ade5961c5ea0102064070fc5a8475ac95c0854de02abeee18b68cd6f6f088efd53098d084533a00d7fa657aca4b4a9ca2a2578726dd3369536b478a65d3670751373c92deb9108a0860e61714f06731fa2634d8be1afebf2ef0e4a2f8cf3d19fef6a05599fd96d1ba019b1808cad0086bbc6325bab37c46d1c78fa604224d462e6dea74018e4ca01d2a04fdb966a77bef14d44dd063b27fe5dede119eae4fd3b15d8cc18f157152b4822a0875d92f629f8cbfb86394de53102bd5722fa668e03905cda48885f59c0268c67a080872257c19fa41ad3d2438d5df245182550040c311a4fa672ed5b9317c4d4b280",
        "0xf90211a0921642d8fe3eda63bdb35574786d302d13035325f8bf0d6d3a8c3cae3e94fc01a0c69e4fe130672466f6f1ff271de3918e6dbab5cd1c7d1241072fdb4bedbc6794a03e9c5775384606d1fb5d51752d7bb68e5fbff5f2adb5670bd41209f24aecad35a0f7df74c3d658a450e472611a916f3ab385651286ba04046347ef6d45523f8e4ca059e16fa89dd83f0b061d1c550099285c2b141a10145d32403fffb5c14fcaa358a0403cfc1e311c4b111456f8158faf24a85634f3a7673316bb41ed262433542287a0ea54484f3937b1f3b3f04335ceae168d0306a66ae9e3b394f311a9ed6a9cb53ba05670a491c93399b0566477fbff559e613f66ecbf382f27dcece929a25643e913a0235f7631d48a5000d371bb6841df9fb09af49951233cc0855b915f538394f275a0656de78e80a6e129a4bc0324a7ec2795f8da08d562cadd7aed20d556a3eee88ba0906453aa798ecc50c62ddf85a915aa851edf86f427905eb2a656db221353466ca0d91539d6d69a83572534bc9bc0de671abea7e2b8e3544c30364e9fb7006663bca0f65f5b85f07bb013d597ebf26172403477aab5ba9b9868a813ac34b1f5f3fe1ea0fa614a267fa6070c51ada654db3fb2bb1c2f4555b4473d5e9d6e3af5f3545e07a0bbefe1363192b11af5da9b38e914d25461b78b59db873c9464b2efb93b750e56a04743830d485df5aea98a1dbf453c1b0377bd56730cd196d7c99bf7203c63697280",
        "0xf90211a00ba3c3a68ac83c86a34e0ea5fa750432ba8d12b648512e86cf64fd1f800ce0d3a0e2643eb5a7cb95040f50891824ac81badf21d2e7b08e84c1395f95c02fe19238a083891ad925d767c2af82c56fb165fda97d8e393b0183ec94a8be1e222f7d257da097d1ac095ae6792f2e4301cde72b0950b2081b1a009ceb34d19a626c18bef3a2a0e92727780a0b9b49831c1ff342f5d55a430840121b0f3468e5ea39debf1d62e4a0f4ba66d5f2e8faf5cdc7440b66d26918056ab1acc49444b48c01c5832eb9f102a0e198570347403b57aa29861810d518aabea911915e15797929a190285a7050b7a04339d7c75c516075901b16d4a629a819a15724908622a3f5e914ecd26c000546a059bec952bcf12fd6a34fa77a3168bb96e3cbf5499be8f68ba05b2a1ec1aa392aa0f048d280d6c5a9032ba7308a859b9f931e8c8d32c960aba8cef23fc328f96279a0405be326ef814bec8197d27d4fd4823fa319be5eae6f4472c7a1e5e497c9e10aa0d84aec724a685a97dc713b507de1bd35cdbf49f49b213fae15275e54866e2371a0b3832b481ecf62542f81e03bb56e388278754ecb0e998cead433e9cfb2dfb6f3a0871a766a8a026ed2a9244ed6671ea997cba5b9454750fc1b36630eee64cc74daa0232de7ab42c7f0d8a276824c8ea5d7fb69d26a32e9a1b78f9ae3a1eafaa07b03a0b69d4beea13c684a69db3046422135737c2b30e197577d81217cd2f943a1b3b980",
        "0xf90211a09442c8002302a291ad481a6fa125287bd3207d3c6e6a50a9a220ef83ea17167fa05ff61d0c2929a593abec1b4d861c6e600bcf482c719835b765fc25e84a99543ba0bc07b99ec4a6b3fa49088710fe8fa99b6c28fa9cb5f7f32b75fce80c517a950fa0c86f006a46f24f21098e91eaac87ba9005ae740b7b29704bb886942109c94b01a0f3d0f73f48f29e4cf74bf4524326114dd199c5c817f7204f8b283cc879717e18a0b0bf7c2da1b0db7ba78147092cc7b72fddf983830f014b9dc149dde619d1eb61a0e16963083f15cf9eb6661aac0b53595437249a879be5a12714358a2bbc67a414a00bed9c9dcd5d7cf07b953fc673b2f0406a3225e924174eb9866604638b19dfa0a055872d1f1925585130b18aaa55e569003794206ce5f947d31e668fb783c699c5a028f095d449084d84a2461f7e9c06fff5ef7f5408b0ba1ef03731c524e367a60ba01534805e0877e5515c6e658cc671d042c00fef4110f75d0f5a6c33ad2472af74a041e65dd0d25042985a956dcbf711da25be6ffb4f899e14c8df6493c96f7fc97aa0980d3987f46813f80886c47ed792b1a1e8fc27092ae5b91f3703f3b45c25e986a0a6f5e09e45cce4251966ef3a9c9cf3af2717b019297046f48bed136281a8c830a06747a59550332abfc418aad82d17fc1e8372be9c7147b8a45c18430c7da9eec6a08098b9c6a055762d8f5ec081fdca991c9776ebf8df1b14e0c94d7f3dd46eacff80",
        "0xf9011180a0f7d07e13736d66b1fee3fb6fec72d2db616882d1ad02fee1e1b21c3d9c3e59098080a004820ca85d9b879c2d099257da6c6bb7365320470bd7d5bd26ea16947aa1b191808080a01f5be1b86cd784852957ff5747b2bad5495df0022d93108c1d76d4d6c66a915fa00309cf26efc1c61285754b0731907ae16c5f306349d840e76ed56d1b71c44f48a00a175ddd92d0987ff07fa6366669a7773862dd025946f63a261b98350c306a9ba058a3167072b5c419c8b3143cad77bdb710e06d6e6477ae1fe62d8b603a7e65eea0d2ad327ee1c14e1ef985a1412aacb1dc89380c66e9fef6167c60d23f4c766b3da02b7cccfcc56273e78d35cc8456ac9b823597922a6359f82b55362a459207470d808080",
        "0xf871a06b6da916964b426119db78ea6b0900c73c8fc8557cf1e7574ad1413d34e35dba80a0c523701fda4ec31012a802fad7c5db21687ab1c2411b934e00a688a7be24909b80a0b3703f2af84380b4bf38f5b0271783c70c6c101a9936b0a8f21980cede7c31af808080808080808080808080",
        "0xf86f9d20badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6b84ff84d018901ae361fc1451c0000a064af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174ba026d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543"
      ],
      "storageProof": [
        {
          "key": "0x6",
          "value": "0x6f05b59d3b200000",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xeba03652222313e28459528d920b65115c16c04f3efc82aaedc97be59f3f377c0d3f89886f05b59d3b200000"
          ]
        },
        {
          "key": "0x2",
          "value": "0x64544dd7",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xe7a0305787fa12a823e0f2b7631cc41b3ba8828b3321ca811111fa75cd3aa3bb5ace858464544dd7"
          ]
        },
        {
          "key": "0x0",
          "value": "0x11d8f8f00cfa6758d7be78336684788fb0ee0fa46",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xf838a0390decd9548b62a8d60345a988386fc84ba6bc95484008f6362f93160ef3e5639695011d8f8f00cfa6758d7be78336684788fb0ee0fa46"
          ]
        },
        {
          "key": "0x13",
          "value": "0x14d1120d7b1600000",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xeca036de8ffda797e3de9c05e8fc57b3bf0ec28a930d40b0d285d93c06501cf6a0908a89014d1120d7b1600000"
          ]
        },
        {
          "key": "0x12",
          "value": "0xde0b6b3a7640000",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080",
            "0xf85180808080808080a095cf8b5a51ce7aa9d43782a2cafb70aa2a6d69322f17390cc141a429c9ba7797a01dc14fd31de0fcdf0f38e604e03349e50cce9866192d18fa298f53649beec4dd8080808080808080",
            "0xea9f3a6a4669ba250d26cd7a459eca9d215f8307e33aebe50379bc5a3617ec344489880de0b6b3a7640000"
          ]
        },
        {
          "key": "0xc",
          "value": "0xa347c391bc8f740caba37672157c8aacd08ac56700",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xf838a03f6966c971051c3d54ec59162606531493a51404a002842f56009d7e5cf4a8c79695a347c391bc8f740caba37672157c8aacd08ac56700"
          ]
        },
        {
          "key": "0xf",
          "value": "0x2e64ac47b6e2fecfcdea35147fe61af9894a06ba6",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xf85180808080808080808080a0b1257206ecadd0e4331d458e5d8cd3ccaaae0ae8385312a46d4f2c79266179248080a0d3345c870762c220f5797fb37592257e195444e4c33c356f44ae33c817964897808080",
            "0xf838a0201108e10bcb7c27dddfc02ed9d693a074039d026cf4ea4240b40f7d581ac802969502e64ac47b6e2fecfcdea35147fe61af9894a06ba6"
          ]
        },
        {
          "key": "0xb",
          "value": "0x64544dd7",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xf85180a035ec97e5b917234746c729a0a7ad2384a8fdc39046d96ae548fa029a15a8413f80a079130f74a1690233120aae4b451d74b2e05cb8776aff460a91a2990ceffc360080808080808080808080808080",
            "0xe7a02075b7a638427703f0dbe7bb9bbf987a2551717b34e79f33b5b1008d1fa01db9858464544dd7"
          ]
        },
        {
          "key": "0x1",
          "value": "0x1064fd9",
          "proof": [
            "0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",
            "0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080",
            "0xe7a0200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6858401064fd9"
          ]
        }
      ]
    },
```

### Account proof parts

Block state root is made from a tree of accounts. Each account has a storage root, which is
made from a tree of storage slots.

Hence the proof Alice gets for a single account has two parts: an account proof and list of storage proofs.

### Degarbling a proof

What is a proof? A list of values that form important parts of a tree. The values have positional information and hash information. When you hash the data in the right way, you can get other hashes
that show the tree matches an expected result. This confirms the data is not modified from how it
was when the whole tree was created. Though now you don't need the whole tree.

A proof is a list of hexadecimal (0-9, a-f) strings. First these are converted to lists of bytes (u8, aka 8-bit unsigned integers, aka 00000000-11111111):
- "0x00" -> 00000000 = 0
- "0x01" -> 00000001 = 1
- "0xff" -> 11111111 = 255
- "0x01ff" -> [00000001, 11111111] = [0, 255]

Hashing is done on these lists of bytes.

Specifically hashing is done using the keccak algorithm. The result of any keccak hash is a 256 bit (32 byte) integer.

Ok, look at the last key (`0x1`) in the storage proof above. The value to prove is `0x1064fd9`. The account storage hash is `0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b`, which is the root for the storage proof.

This is a 64 character hex string, or by other measurements:
- 32 bytes
- 256 bits (a number with 256 0s and 1s)
```py
>>> bin(0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b)
> 0110010010101111000000110100101100111011111111110000010011101001010110110101110110101011001000010110100111110111111010100011101010010010101010101111101111111010011010001111101101000000111100010110110101111101000101100110010010101000111101000001011101001011
```

How do we get to the root `0x64af...174b` from key:  `0x1` value: `0x1064fd9`?

```json
"proof": [
"0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",

"0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080",

"0xe7a0200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6858401064fd9"
]
```

By hashing the first element, we obtain the root.
```
keccak(0xf901...3080) = 0x64af...174b
```
So the proof has the following form:
- root
    = is hash(first node)
- first node
    contains hash(second node)
- second node
    contains hash(third node)
- third node
    - data

### Decoding the first node

Let's check our understanding by manually decoding the RLP-encoded trie nodes, starting with
the first one.

It starts with `0xf9`. Looking at: https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/#rlp-decoding, one can see that this is a long list.

"the data is a list if the range of the first byte is [0xf8, 0xff], and the total payload of the list whose length is equal to the first byte minus 0xf7 follows the first byte, and the concatenation of the RLP encodings of all items of the list follows the total payload of the list"

That is somewhat convoluted. In other words:
- If the start is [0xf8, 0xff], its a long list
- The length (bytes) of the list is given, followed by the list itself
- the bytes used to represent the length is `first - 0xf7`.
    - So we have 0xf9 - 0xf7 = 0x2 bytes for the length.
- The list comes after that, and will itself be RLP-encoded.

```sh
0xf90111a0ded...
// long list. Next 2 (0xf9-0xf7=0x2) is length), length, data - remaining 546 chars,
"0xf9                                           0111    0aded..."
```
Thus, the data is 273 bytes (0x111), which is the same as 546 hex characters. The RLP encoded
data in this list starts with `0xa0ded8...`

So following the decoding rules for this first item, 0xa0 indicates that it is a long string. The
length of the string is 0xa0 - 0x80 = 0x20 (decimal 32). So a 32 byte string.


```
0xded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b5
```

Visible within the original string:
```
0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e...
          ^>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>^
```
Then we have something interesting! The next value after this string is 0x80. We can see from the
decoding guide that this is a string with a length of 0x80 - 0x80 = 0x0. So an empty string. Then the next item starts with 0xa0, same as before (a 32 byte string). Repeating the process for all the data, we get:

```
0xf90111 (273 byte list)
a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b5

80 (0x80 is an empty string)

a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa

80

a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d

80

a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b

80

a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b37

80

80

a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f8

80

a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c

80

a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b5730

80
```

Which is indeed a list of 17 items. Knowing that a keccak hash is 32 bytes we
can see that this is a list of hashes and empty strings as follows:

```
[hash, null, hash, null, hash, null, hash, null, hash, null, null, hash, null, hash, null, hash, null]
```


Looking at Merkle Patricia Tries https://ethereum.org/uk/developers/docs/data-structures-and-encoding/patricia-merkle-trie/#basic-radix-tries, we can see that
a branch node has 16 children and 1 value, which makes 17 total. This particular piece of data has
8 children hashes, 8 children that have no hash and a value that is empty.

In summary we have an account proof. The account proof includes proofs for individual storage
slots. Looking at one such storage proof, we found it consisted of three pieces of data encoded
with RLP. We decoded the first part and showed that it is a merkle patricia trie branch node
with 8 of 16 children populated and with no value.

Before
```json
"proof": [
"0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080",

"0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080",

"0xe7a0200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6858401064fd9"
]
```

After:

```json
"proof": [
// First node: A branch node
[
    "0xded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b5",
    "",
    "0xe423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa",
    "",
    "0xb234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d",
    "",
    "0x42a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b",
    "",
    "0x3b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b37",
    "", "",
    "0x5f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f8",
    "",
    "0xffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c",
    "",
    "0x3f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b5730",
    "",
],
// todo: decode RLP
"0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080",
// todo: decode RLP
"0xe7a0200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6858401064fd9"
]
```
### Decoding the second node

Knowing more about RLP, one can now see that those long runs of `808080`'s are probably going
to be a run of empty strings in a list. Let's decode them anyway.

Target:
```
0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080
```

Consult the guide: 0xf8 is a long list.

```sh
0xf85180a0...
/ long list, length of binary data 81 bytes long, RLP data
"0xf8        51                                   80a0..."
```
Full decoding:
```
f8 51

80

a0 20ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094

80 80 80 80 80 80 80 80 80

a0 5ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b607

80 80 80 80 80
```

[null, hash, 9-nulls, hash, 5-nulls]
### Decoding the last node

Let's try with the last part of the proof:

```json
"0xe7a0200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6858401064fd9"
```
RLP decoding `0xe7` indicates a short list with length 0xe7 - 0xc0 = 0x27 (decimal 39).

```
0xe7

a0 200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6

85 8401064fd9
```

0x85 is a string with a length 0x85 - 0x80 = 0x5 (5 bytes).

The decoding is finished, leaving the string "0x8401064fd9".

Note that the value we are trying to prove is 0x01064fd9. If we treat
the string as another RLP encoded value, see that it
starts with 0x84, which is a string of length 0x84 - 0x80 = 0x4. Hence "0x01064fd9"
is derived by RLP-decoding the value of this leaf.

### RLP-Decoded proof
```json
"proof": [
// First node: A 17-element (branch) node
[
    "0xded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b5",
    "",
    "0xe423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa",
    "",
    "0xb234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d",
    "",
    "0x42a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b",
    "",
    "0x3b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b37",
    "", "",
    "0x5f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f8",
    "",
    "0xffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c",
    "",
    "0x3f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b5730",
    "",
],
// Second node: A 17-element (branch) node
[
    "",
    "0x20ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094",
    "", "", "", "", "", "", "", "", "",
    "0x5ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b607",
    "", "", "", "", "",
]
// Third node: A 2 element node (leaf or extension)
[
    "0x200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6",
    "0x01064fd9"
]
```
### Getting the value from the proof
Now the proof is decoded. So we can see that it is a list of merkle patricia
trie nodes and that the last element has a value of 0x01064fd9.

Recall that this proof is for the following key-value pair:

```
"key": "0x1",
"value": "0x1064fd9",
"proof": [
    "0xf901...
]
```
So the value being proven is indeed located in the proof.

### Verifying the proof hashes

Now we have a trie that links the value of interest to the storage root of the account.
The next step is to recreate the trie, hash it and confirm that the root matches the
expected root (`0x64af...174b`).

So far, the proof has three parts [ upper, middle, lower].

Let us hash the lower RLP-encoded part:
```
hash = keccak(lower)

= keccak(0xe7a0200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6858401064fd9)

= 0x20ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094
```
This is one of the hashes (index=1) in the middle part of the trie!

Hashing the middle:
```
hash = keccak(middle)

hash = keccak(0xf85180a020ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094808080808080808080a05ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b6078080808080)

hash = 0x5f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f8
```
This is one of the hashes (index=11) in the top part of the trie!

Let's hash the top part of the trie.

```
hash = keccak(top)

hash = keccak(0xf90111a0ded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b580a0e423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa80a0b234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d80a042a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b80a03b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b378080a05f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f880a0ffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c80a03f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b573080)

hash = 0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b
```

Have we seen this value before? Yes, it is the storage hash for account `0x431a...528b` at block
`17_190_873`:

```json
"0x431a2b6ff34a865b6cd7f498e371e2510691528b": {
      "balance": "0x1ae361fc1451c0000",
      "codeHash": "0x26d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543",
      "nonce": "0x1",
      "storageHash": "0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b",
```
### Showing the structure of the proof
Now we know the trie has this structure:
```
// hash of RLP data: keccak("0xf901...3080")
- Storage_hash "0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b"
- Proof
    - "0xded8a060b7117ab6a09257ca5cee2731149e80e7a6577f58e68ef484c4d8d0b5",
    - "",
    - "0xe423e636e10515529e3fde202ab92c1851d62225bcae4fae2e0995cc157477aa",
    - "",
    - "0xb234b10f479bd0705c9e48d37f40948aee77245d556d13323e49feb71ca39a9d",
    - "",
    - "0x42a24800822e8168aa77aa8adcf2fec5fbdcfbc68e3df2546fe62fff8b8b547b",
    - "",
    - "0x3b868f9c0c16a531dbc02db219e46e1330b5f79ec5de1b6f3b1eff4e38134b37",
    - "",
    - "",
    // hash of RLP data: keccak("0xf851...8080"), AKA middle
    - "0x5f604036e8750803fa23122a5aad01399df15970e8ec46869d68b4b528b4b4f8",
        - "",
        // hash of RLP data: keccak("0xe7a0...4fd9"), AKA bottom
        - hash "0x20ea929389f29bef186d37d8de415dcdf5a7b52b80377bb6b0559317ae2d0094",
            // Leaf node, even remaning path, remaning path
            - "0x200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6",
            // value to be proven
            - "0x01064fd9"
        - "",
        - "",
        - "",
        - "",
        - "",
        - "",
        - "",
        - "",
        - "",
        - "0x5ac25e7a1edb36b16e4fdef56cb7daa98a9e6e58fd374cc591d6f288ee84b607",
        - "",
        - "",
        - "",
        - "",
        - "",
    - "",
    - "",
    - "0xffba59b8b86dc6540e6841344d840708269e8840dcfa38c079f7f8a93e15c04c",
    - "",
    - "0x3f4932fbb782ee71a1161c8d3021298df1a9d1d86bcbd48a0d657945ab8b5730",
    - "",
```

An overview of this naive pattern:

- Storage value
- Account root
- Account proof (list of RLP encoded data, [A, B, ... C])
    - A (Top of tree / near root). `hash(A) == account_root`
    - B (Lower in tree), `hash(B)` will be present in A.
    - ...
    - C (Bottom of tree / near branches), `hash(C)` will be present in B
        - `hash(storage_value)` will be present in C.

So, we have
1. hashed the value for this storage slot, and found that hash in C
2. hashed C and found it in B
3. hashed B and found it in A
4. hashed A and confirmed it matches the storage root.

Note that we have not yet proven the storage root, once this is done, we can verify
that we can obtain a hash that matches the state root present in the block header for
block `17_190_873`.

## Proper navigation: Using the path

There is a system for determining the structure of the trie that we have so far ignored.
That is the path.

A path is a method to walk from the first node to the final node. The path is defined
as the hash of the key. This allows paths to be very different, even when keys are similar.
This helps with managing database accesses and comes under a topic called "tree balancing".

Let's look at the path for the key (0x1) in our example. We hash the key, representing
it first as a 32 byte value.
```
path = keccak(key)
path = keccak(0x0000000000000000000000000000000000000000000000000000000000000001)
path = 0xb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6
```

This path describes 64 "steps" to traverse along the trie, starting from the root.

Step 1: take path 'b' (the element with index 11 (0xb))
Step 2: take path '1' (the element with index 1 (0x1))

This explains why branch nodes have 16 elements (hexadecimal 0xf = 16) to choose (and a 17th
element that is never used).

- Root
- Node 1: 12th item
- Node 2: 2nd item
- Node 3: follow remaining path (0e2d...)

### Path navigation at leaves

We can see that a leaf has two items. The second item is the value. The first
contains the remaining path (0e2d...).

However it also contains a description of the kind of node (leaf or extension) and the
whether there is an even or odd number of steps left in the path traversal.

As paths are walked one hexadecimal character (one nibble) at a time, a node may have
an odd number left. As two nibbles are needed for a byte (0xf -> 0x0f), this information
is included to avoid ambiguity [https://ethereum.org/uk/developers/docs/data-structures-and-encoding/patricia-merkle-trie/#specification](https://ethereum.org/uk/developers/docs/data-structures-and-encoding/patricia-merkle-trie/#specification).


So, looking at the final node once more:

```
[
    "0x200e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6",
    "0x01064fd9"
]
```
The first byte 0x20 tells us two things:
- Leaf node
- Even number of nibbles (characters) left

So, we can then ensure that the remaining path (0e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6) completes
the traversal.

### Exclusion proofs
If the path did not match exactly, then this would be conclusive proof that the key
was not part of the trie. In other words, when a proof ends up in a dead end it is
a good proof that the key does not exist in the data.

It is a successful exclusion
proof one can say "This confirms the key is absent, and that the value to be used is the default
value". The default value might be `0x0` as is the case for storage values.

Exclusion proofs can end on any sort of node. The moment a path diverges from the expected path,
the proof becomes an exclusion proof. If the claimed value is not the default value, then
the proof is invalid. Someone has tried to claim that a key exists and has a certain value,
but in reality the key does not exist (at that moment in time, e.g., at block 17_190_873).

### Extension nodes

When a the trie is being constructed or modified, two keys might have very similar paths.
Suppose there is 10 nibbles that are the same at the start of the path. Rather than
having 10 branch nodes one can instead make a single extension node. The extension node
consists of the common path (the 10 nibbles), and the next node (a branch node for the 11th
nibble where the paths diverge).

- leaf: (remaining path, value)
- extension: (common path, next node)

## Similarity amongst storage proofs

Looking at the separate storage proofs for this account, we see that the first element of
each proof is the same. This makes sense because they all represent the top of the trie
and need to be hashable to get the storage root. The subsequent parts of each proof may
be also be the same (e.g., the second part of the proofs for keys 0x10 and 0x12 are also
identical). This happens whent the data is in a similar part of the trie.

## Automation

So to automate the process we can do as follows:

- Go through each storage proof
- Hash the key to get the path to traverse
- Go through each part of the proof
- Hash the RLP data and check it matches the node above (or root if first)
- RLP decode
- Traverse node items, according to the path to get the next hash.
- If the traversal doesn't match, it is an exclusion proof, so check the claimed value is the default value
- If the traversal matches, then check the final leaf has the claimed value.

This must be done for account proofs and for storage proofs. An account proof should be checked
before a storage proof because storage proof roots are located inside accounts. If
the account proof is invalid, then the storage proof means nothing.

## Account proof

Where the storage proof decoded above had 3 parts, the account proof has 9.

A trie adds more levels when two items have the same prefix and need to be differentiated.
For example, the node prefixes 0xabcd and 0xabc1 share a common prefix 0xabc. If there
are many entries in the trie at each level: (0xa..., 0xab..., 0xabc...), then a new level is added
to enable differentiation between 0xabcd and 0xabc1.

So, an account proof with a depth of 9 tells us that there are many more accounts than there are
storage values for this account.

As with the storage proof, the account proof consists as a list of elements. Hashing the first element:
```
hash = keccak(top)

hash = keccak(0xf90211a0cddf489c063d666cac6218e7059d42beaca7ed5eb5fe8d6fc0b09353c25db199a02ba7f8e5426258da2a51327c018db462ccb059da51eb9bdd84da4cc241298eada0b50c1cc0ce7cd5195ad856d3003dbb8c24852d4f4d6e77151548326cd416fc71a08c080f1bd59931fe8d32d275b678dfe51ac6adb6dcb15ebf516c4c81784d006da0318f22a4e670a7c8e577f71ae7720944d989d661d25d6e39c2ee3f69c50bb82fa07f5901b077738a0629cb3c0350673cf3b3dcea06d6475434575f605beafb4a40a060fddfe2f6d0ab542001878eb89a5d9308033a49522f207f5f0c1fd6a9938544a04d1779e5d5bb0673ff8f76f1dd0952cc98e33e74bad35cdc508ef60d8203af97a0e97f4d0739dc2fc64a84defa064f6200519f6cce7a350711ffc00c4fa9e355bba0e64de506951857c5d5faa34ae91173e2739c0185102367685ede543bb1237277a03551c85d2d49127e70c4eda38184046eac4c9bef5a5a8a08e9722e57aeaa1cd7a01eb2c01d2ff70b62951b30c0a78affcfb6d6416865ff72b4deb7b6a32aca5a3ca096911875c66150e05274005df18c37c33679d960260d82e494941b441ed11082a0f1d58ab8af4d60fa255221c35cbceea2c07eab25cb9e6584f7b1f3d162e370baa0674a08ae583eaa6011049430c8a442ddd8abeb330e0a162636fd6b642fd43535a056061ecbc123c7a4aee4536b853c707173437e270a0d3c9815231b5b7c375c4180)

hash = 0x38e5e1dd67f7873cd8cfff08685a30734c18d0075318e9fca9ed64cc28a597da
```

The state root present in block `17_190_873` header is
`0x38e5e1dd67f7873cd8cfff08685a30734c18d0075318e9fca9ed64cc28a597da`, which matches the
computed hash from the proof.

### Account proof path

The path for accounts is defined by the account address
```
path = keccak(address)
path = keccak (0x431a2b6ff34a865b6cd7f498e371e2510691528b)
path = 0x9d476dd4badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6
```
So the first branch node will follow the item with index 9, and the leaf will end with
some amount of `...56f5f4d9c6`.

### Account proof leaf


Let us examine the last element in the account proof:

```
0xf86f9d20badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6b84ff84d018901ae361fc1451c0000a064af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174ba026d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543
```
Decoding the RLP, we note that 0xf8 is a long list. There is 1 (0xf8 - 0xf7 = 0x1) byte of data for
the length of the binary data. That length is 0x6f (decimal 111). Then we have data from 0x9d20... onwards (222 hex characters, which is indeed 111 bytes.)
```
0xf86f9d20...
// long list, length of binary data, RLP data
"0xf8        6f                     9d20..."
```

Decode the whole thing:
```
// long list with 0x6f (111 bytes of data)
0xf8 6f

// a 29 byte string
9d 20badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6

// a string with length 79 bytes will follow
b8 4f

// Another list with length 0x4d (77 bytes /  144 chars = remaining data)
f8 4d

    // A byte with value 0x1
    01

    // A string of length 0x89 - 0x80 = 0x9 (9 bytes)
    89 01ae361fc1451c0000

    // A 32 byte string
    a0 64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b

    // A 32 byte string
    a0 26d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543
```

Result:
```
[
    "0x20badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6",
    "4f",
    [
        0x01,
        "0x01ae361fc1451c0000",
        "0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b",
        "0x26d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543",
    ]
]
```
0x20badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6 is the encoded path: 0x20 indicating
a leaf node with even remaining nibbles and `badcb...` is the remaining path.

An account object is defined as: `[nonce, balance, storageHash, codeHash ]`,
so we have:

```json
{
    "nonce": "0x1",
    "balance": "0x1ae361fc1451c0000",
    "storageHash": "0x64af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174b",
    "codeHash": "0x26d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543",
}
```

Let's hash this leaf node:
```
hash = keccak(lowest)

hash = keccak(0xf86f9d20badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6b84ff84d018901ae361fc1451c0000a064af034b3bff04e95b5dab2169f7ea3a92aafbfa68fb40f16d7d1664a8f4174ba026d864293ab640d5742c797a7a66b01cfea8120ebc91dcb773099e5c7dfbb543)

hash = 0xb3703f2af84380b4bf38f5b0271783c70c6c101a9936b0a8f21980cede7c31af
```

Now we can see if this hash appears in the level above (one step toward the root) in the account proof (0xf871a0...808080).
```
]
    "0x6b6da916964b426119db78ea6b0900c73c8fc8557cf1e7574ad1413d34e35dba"
    "",
    "0xc523701fda4ec31012a802fad7c5db21687ab1c2411b934e00a688a7be24909b",
    "",
    "0xb3703f2af84380b4bf38f5b0271783c70c6c101a9936b0a8f21980cede7c31af",
    "" * 12
]
```
Yes, it is present at the sibling with index=4 in the level above, which matches the path traversal. We will skip manually verifying the remainder of the nodes for brevity. The
structure will be as follows for the 9 nodes in the proof:

```
// Path in first 7 branch nodes
9 d 4 7 6 d d

// Path we checked in second last (branch) node
4

// Path we checked in the leaf node
badcb702c3a2f2529bc0456f82f8dbda5669e6acbd3fad56f5f4d9c6
```

As the account containes the storage root, the storage proofs are now also verfified.


### Conclusion

This concludes the manual interrogation of proof formats.

The overall process is best described as:

1. Hash the key (storage key or account address) to get the path
2. Start with the first node in a proof
3. RLP decode the node
4. Chose to walk down a particular item in the node
5. Walking is done by taking a nibble/character at a time
6. Check that walking the path results in the final expected value
7. Check that all parents contain the hash of the child node along the walk
8. If the path diverges, it is proof that the key is not in the trie.

Proofs are interesting because they prevent adversaries for fabricating data.
This allows strangers to send data that is true. This is useful to explore the
history of the Ethereum chain, as is explored in the next post.

Continue on to [tracing](tracing.md) without an archive node.