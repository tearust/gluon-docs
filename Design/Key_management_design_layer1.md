

### Register Gluon wallet account

Browser: send the hashed nonce to layer1

Layer1: store the hashed nonce in account info map 

```rust
// called by browser
pub fn send_nonce {
  origin,
  hashed_nonce Cid,
}
```



App: send registration application to layer1

```rust 
// called by app
pub fn send_registration_application {
  origin,
  nonce Cid,
  app_pk AccountId,
  nonce_signature Cid,
}
```



Layer1: verify the signature and compare the nonce hash of app and  browser. if passed, an event will be fired.

```rust
registrationApplicationSucceed(BrowserPublicKey, AppPublicKey)
```



### Generate BTC 2/3 MultiSig Account

Browser: send hash of nonce, hash of task detail, and signature to layer1.

Layer1: verfiy the signature and store the hashed nonce and hash of the task detail.



App: send nonce, signed nonce using AppSecKey,  delegator and task full detail to layer1.

Layer1: verify signature, compare the nonce hash of browser and app,  compare the task hash of browser and app.



**①layer1 mthods**

```rust
// called by browser
pub fn browser_generate_account {
	origin,
  hashed_nonce Cid,
  task_hash Cid,
}

// called by app
pub fn generate_account_without_p3 {
	origin,
	nonce Cid,
	nonce_signature Cid,
	delegator_tea_id: TeaPubKey,
  key_type: Cid,
  p1 Cid,
	p2_n: u32,
	p2_k: u32,
}

// called by app
pub fn generate_account {
	origin,
	nonce Cid,
	nonce_signature Cid,
	delegator_tea_id: TeaPubKey,
  key_type: Cid,
  p1 Cid,
	p2_n: u32,
	p2_k: u32,
  friends_secret Cid, //secret of your friends
  p3_k: u32,
}

```

```rust
// called by facade
pub fn update_generate_key_without_p3_result(
	origin,
	task_id: Cid,
  p2_deployment_ids: Vec<Cid>,
)

// called by facade
pub fn update_generate_key_result(
	origin,
	task_id: Cid,
  p2: Cid,
  p2_deployment_ids: Vec<Cid>,
  p3: Cid, // if len == 0 means 2/2 multi signature
  p3_deployment_ids: Vec<Cid>,
)
```

**②facade data structure**

```json
{
  "KeyGenerationData": {
          "fields": {
            "keyType": {
              "rule": "required",
              "type": "string",
              "id": 1
            },
            "n": {
              "rule": "required",
              "type": "int32",
              "id": 2
            },
            "k": {
              "rule": "required",
              "type": "int32",
              "id": 3
            },
            "delegatorTeaId": {
              "rule": "required",
              "type": "bytes",
              "id": 4
            }
          }
        },
 "GenerateKeyResponse": {
          "fields": {
            "taskId": {
              "rule": "required",
              "type": "string",
              "id": 1
            },
            "p2_dataAdhoc": {
              "rule": "required",
              "type": "KeyGenerationData",
              "id": 2
            },
            "p3_dataAdhoc": {
              "rule": "required",
              "type": "KeyGenerationData",
              "id": 3
            },
            "payment": {
              "rule": "required",
              "type": "string",
              "id": 4
            }
          }
        },
 "GenerateKeyWithoutP3Response": {
          "fields": {
            "taskId": {
              "rule": "required",
              "type": "string",
              "id": 1
            },
            "p2_dataAdhoc": {
              "rule": "required",
              "type": "KeyGenerationData",
              "id": 2
            },
            "payment": {
              "rule": "required",
              "type": "string",
              "id": 3
            }
          }
        }
}
```

**③layer1 event** 

```rust
KeyGenerationRequested(Task_hash, KeyGenerationData)
UpdateGenerateKey(Task_hash, KeyGenerationResult)
```



###In common usage cases

Browser: request gluon to sign using P2, send signature and transaction data to layer1.

Layer1: verify the account of browser and store the transaction detail temporarily to the block chain.



App: sign transaction using P1 and send the signature to layer1.

Layer1: verify app account and store the signature temporarily to the block chain.



Layer1：if received P1 signature from app and  received transaction data from browser， start task to sign transaction using P2 and remove signature and transaction data from the block chain.



**①layer1 methods**

```rust
pub type TxData = Vec<u8>

// called by browser
pub fn sign_tx(
	origin,
	task_id: Cid,
	data_adhoc: TxData,
	delegator_tea_id: TeaPubKey,
)

// called by app
pub fn update_p1_signature {
  origin,
  p1: Cid,
  p1_signautre: TxData,
}

// called by facade
pub fn update_sign_tx_result(
	origin,
	task_id: Cid,
  P2: Cid,
  P2_signature: TxData,
)
```

**②Facade data structure**

```json
{
"SignTransactionResponse": {
          "fields": {
            "taskId":{
              "rule": "required",
              "type": "string",
              "id": 1
            },
            "keyTaskId": {
              "rule": "required",
              "type": "string",
              "id": 2
            },
            "dataAdhoc": {
              "rule": "required",
              "type": "bytes",
              "id": 3
            },
            "delegatorTeaId": {
              "rule": "required",
              "type": "bytes",
              "id": 4
            },
            "payment": {
              "rule": "required",
              "type": "string",
              "id": 5
            }
          }
  },
"UpdateSignTransactionResult": {
          "fields": {
            "taskId": {
              "rule": "required",
              "type": "bytes",
              "id": 1
            },
            "signature": {
              "rule": "required",
              "type": "bytes",
              "id": 2
            }
          }
 }
}
```

**③layer1 event**

```
UpdateSignTransactionResult(taskId, signature)
```

