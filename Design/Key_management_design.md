# Overall

![](./1.svg)

Let's assume Alice is our client. She is going to create a BTC wallet that managed by Gluon.
In order to get disaster recovery, she assign her three friends Bob, Charlie and Dave to be her recovery accounts.

Alice needs to have a TEA Layer1 plugin (Polkadot extension) on her browser. She also needs to install Gluon mobile app on her phone.

Gluon will create a 2/3 Multisig BTC account. The 3 keypairs are called P1, P2 and P3.

P1 is sent to Alice's mobile phone immediately. Gluno won't have any copy of P1 secret key

P2 is managed by Gluon under Alice's control.

P3 is managed by Cluon under Friend Recovery Smart Contract Control (FRSC). 

Because this BTC account is 2/3 Multisig. Any 2 of P1 P2 P3 can pass authentication.
## Alice register Gluon wallet account

```puml
@startuml
autonumber
box
actor Alice
actor AliceBrowser
actor AliceGluonApp
endbox
box
actor TeaLayer1
actor GluonLayer2
endbox
Alice -> AliceGluonApp: Download APP and install on her phone.
AliceGluonApp -> AliceGluonApp: Generate RSA keypair. Pub key as AppPubKey. App store AppSecKey in a secrete place

Alice -> AliceBrowser: Download and install Polkadot Extension. Create a new account of TEA Layer1.
AliceBrowser -> AliceBrowser: Start pairing mode. Generate a random nonce at Browser (or extension) in QR code format. 
note right
The QR code include
- Nonce
- Alice browser's public key. Also called Alice's layer1 account ID, or just accountID.
end note
AliceBrowser -> TeaLayer1: Send the hashed nonce to layer1. 
TeaLayer1 -> TeaLayer1: Stored the hashed nonce in Alice account info map. 

AliceBrowser -> AliceGluonApp: Alice use AliceGluonApp's camera to scan such a QR code got the nonce and AliceBrowser's public key (this is Alice layer1 account ID)

AliceGluonApp -> TeaLayer1: Send registration application 
note right
including: 
- nonce, 
- AppPubKey, 
- Alice Browser public key (this is also Alice's accountID)
end note

TeaLayer1 -> TeaLayer1: Hash the nonce, then compare with stored hashed nonce in Alice' account info map.
note right
If hash does't match. Drop this request. Someone is trying to imperson Alice's phone.
If match, store AppPubKey to Alice's account info map. Use this pubkey for future communication mobile app verification
The nonce is for one-time-use. It can be removed from layer1 storage now.
end note

AliceBrowser -> TeaLayer1 : Query updated account information. 
note right
We should see a mobile app has been paired with Alice's account. The AppPubKey is shown.

end note
@enduml
```

Only one mobile app can be paired with Alice's account at any time. If Alice want to update her phone, she need to run the Transfer task to assign new phone and disable old phone.

Once Alice' mobile app is paired with Alice' account. Only Alice's mobile app can send message to both Gluon layer1 and layer2. Everytime Gluon will use a nonce to verify AppPubKey signature to prevent man in the middle attack.
## Generate BTC 2/3 MultiSig Account
```puml
@startuml
autonumber
box
actor Alice
actor AliceBrowser
actor AliceGluonApp
endbox
box
actor TeaLayer1
actor GluonLayer2
endbox
=== Alice submit Generate BTC Account task request ===
Alice -> AliceBrowser: Run Generate BTC Account webapp in AliceBrowser
AliceBrowser -> AliceBrowser: Create a QR code with information: task_name: GenerateBTC, nonce: 1234(random number)
AliceBrowser -> TeaLayer1: Hashed nonce.
TeaLayer1 -> TeaLayer1: Record hashed nonce for one-time-verification use. Temporary
AliceGluonApp <- AliceBrowser: Alice use mobile app to scan the QR code. Use fingerprint to confirm.
AliceBrowser -> TeaLayer1: Send confirmation including: nonce, signed nonce using AppSecKey, task_name
TeaLayer1 -> TeaLayer1: Verify 1. AppPubKey verify signature 2. Hash the nonce compare with Browser uploaded hashed nonce.
TeaLayer1 -> TeaLayer1: Generate a Generate BTC task on behalf of Alice
== Generate BTC Account ==


@enduml
```
## In common usage cases

In common usage, only P1 and P2 are used to sign a transaction.

Typically Alice can start a transaction from a web browser. Once a transaction is created (for example, sending $100 to Eve).  

Alice use her browser's Polakdot Extension to sign a signature request (SigReq) of this transaction. Note: This is a Signature Request sending to Gluon, not a signature of this transaction. Gluon receives this SigReq, save it in Task Pool but not continue until P1 (Mobile app) to sign first. After few seconds, her mobile Gluon app receive a push notification. Alice use her fingerprint (or Face recognization depends on her phone settings) to unlock the Gluon app. Gluon app will show the detail of transaction, in our case "Sending $100 to Eve). Alice re-confirm the transaction then click sign. Alice's mobile phone has P1 secret so that the app can sign this transaction. P1 is completed. This P1 signature also send to Gluon. Gluon received P1 Signature and verify. If this signature from a registered phone all and verify successfully, Gluon will conintue to get P2 signature using Sharmir Secret Sharing Schema. Once P2 is signed, both signature send to BTC.

![](./2.svg )

```puml
@startuml
autonumber
box
actor Alice
actor AliceBrowser
actor AliceGluonApp
endbox
box
actor TeaLayer1
actor GluonLayer2
endbox

actor BTC

Alice -> AliceBrowser: Create a tx
AliceBrowser -> TeaLayer1: Request Gluon to sign using P2
TeaLayer1 -> TeaLayer1: Verify AliceBrowser P2; Create SigReq task
TeaLayer1->GluonLayer2: Find Executor to process SigReq task
GluonLayer2->AliceGluonApp: Push notification. Wake up with a session_id. At meantime start FindProvs to save time
AliceGluonApp -> GluonLayer2: Query session_id
GluonLayer2 -> GluonLayer2: Verify AliceGluonApp; 
GluonLayer2 -> AliceGluonApp: SigReq from AliceBrowser
AliceGluonApp -> AliceGluonApp: Show tx detail on mobile screen. Request Alice to confirm using fingerprint
Alice -> AliceGluonApp: Confirm by unlock fingerprint
AliceGluonApp -> GluonLayer2: Signature of Tx using P1
GluonLayer2 -> GluonLayer2: Verify Signature from AliceGluonApp. If pass, start P2 signature process (SSS)
GluonLayer2 -> BTC: Send 2 signature satisfied 2/3 MultiSig
@enduml
```

## Transfer the P1 to a new phone

We do not recommended our client to backup mnemonic passphrase. If client cannot manage the mnemonic correctly, it may cause secret leaking. Although if Alice leak P1 only may not cause any asset loss, it increase the risk since there is only P2 or P3 to protect her assets. However, we allow client to upgrade there phone so that the secret can be transferred from old phone to new phone at a much lower risk. That means a one-time display mnemonic. 

In order to discourage Alice display and backup her mnemonic, we would force Alice to install Gluon app on her new phone before she can display mnemonic on her old phone. When new phone registered successful, Gluon will allow old app show the mneonic in text or QR code once. At this same time, revoke old app's registration. Any new transaction sent from old app won't be accepted. 

Alice can use her new phone's Gluon app to scan the QR code immediately. New Gluon app will recover the P1 secret. Once done, sending successful signal to Gluon. Gluon will request old Gluon app to destroy secret also disable its access forever. Any information sending from the old app won't be accepted from now on.

Alice needs to be very careful when transfer P1 from old phone to new phone. She needs to make sure no one can see the QR code or mnemoonic during the transfer. 

![](transfer_p1_between_phones.svg)



```puml
@startuml
autonumber
box
actor Alice
actor AliceBrowser
actor OldAliceGluonApp
actor NewAliceGluonApp
endbox
box
actor TeaLayer1
actor GluonLayer2
endbox

Alice -> NewAliceGluonApp: Download and install GluonApp to Alice's new phone. Generate new AppPubKey AppSecKey
NewAliceGluonApp -> TeaLayer1: Generate a random nonce; Submit a transfer p1 request 
note right
including:
- AppPubKey of the new app
- Hash of Random nonce
- signature of nonce using AppSecKey
end note

TeaLayer1 -> GluonLayer2: Execute transfer task
GluonLayer2 -> OldAliceGluonApp: Push notification
OldAliceGluonApp -> GluonLayer2: Query
GluonLayer2 -> OldAliceGluonApp: Notify Alice that you have a new phone asking to transfer p1 from the current phone
NewAliceGluonApp -> Alice: Show the QR code of nonce
Alice -> OldAliceGluonApp: Scan the QR code on New Phone screen, the nonce enter old phone.
OldAliceGluonApp -> OldAliceGluonApp: Show the notice, let Alice fingerprint confirmation. 
OldAliceGluonApp -> GluonLayer2: Send hashed nonce, waiting for confirmation
GluonLayer2 -> OldAliceGluonApp: Confirm the hash matches
OldAliceGluonApp -> Alice: Show P1's QR code and mnemonic phrases
Alice -> NewAliceGluonApp: Using new phone to scan old phone's QR code.
NewAliceGluonApp -> NewAliceGluonApp: Store the P1 into secured place. 
NewAliceGluonApp -> GluonLayer2: Send notification of successful transfer.
GluonLayer2 -> TeaLayer1: Replace OldAliceGluonApp's AppPubKey with NewAliceGluonApp's AppPubKey. From now on, only new app can communite with Gluon
GluonLayer2 -> OldAliceGluonApp: Destory P1 and wipe out memory. Stop screen display. Disable everything
GluonLayer2 -> NewAliceGluonApp: Confirm transfer successfully.

@enduml
```

## If Alice lost both her phone and computer (p1 and p2)

The FRSC (Friend Recovery Smart Contract) controlled P3 is only used when Alice lost her phone AND her layer1 account secret key. Alice is lucky, this is just because she lost the key, not leaking. No one have both her P1 and P2 yet. If someone have her P1 and P2, he can take over Alice's assets.

This is a very rare case and if this happened in other blockchain projects, this could mean that Alice will lose her assets forever. But we can still use Friend recovery to get the P2 back to her new generated layer1 account, so that her assets are saved.

![](3.svg)

![](4.svg)

# Replica and Shamir Secret Sharing
![](replica_repin.svg)
