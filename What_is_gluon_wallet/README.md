## You say “I need a hardware wallet”, I heard “I need some way to protect my private key”
Only in Bitcoin, according to [this article](https://news.bitcoin.com/analyst-1500-bitcoins-lost-every-day-less-than-14-million-coins-will-ever-circulate/) 1,500 Bitcoins Lost Every Day. 

![](../res/blog/1_SO8ID-UqYDVWXzYFjUQ4ow.png)

1,500 BTC EVERY DAY! If we convert to today’s BTC price($32,800 on Jan 2nd, 2021), it is 49 Million USD a day, or $34167 every minute! When you spend 3 minutes reading this page, do the math you will know how much has been lost. You do not want to be the victim, neither do we. 


## Gluon is NOT "Yet another hardware wallet", it is actually a Taas(Trust_as_a_Service) protecting your private keys

So far hardware wallets are the best way to protect your crypto assets but they still have caveats that put you at risk. What if there is a better way to protect your crypto assets? 

Gluon is not a wallet but a Trust-as-a-Service. The root of trust comes from thousands of hardware wallets although you do not own them. You do not need to trust the owners of those hardware wallets, because they have no access to your secret, even you do not?

## How does it work?

Gluon protect your crypto assets in major blockchains which supports MultiSig. There would be 3 private keys. One key is held on your smart phone. Although it was protected by your biometric lock, it is still the weakest one of three. All other two are scrambled and distributed to unknown TEA nodes. Every TEA node is technically  a hardware wallet but can do more. No one, including you (the owner) and us (the system builder) and the miner (the system runner) has the knowledge of where your keys are located. The keys can yet be rebuilt and sign your blockchain transaction anytime you want. One of these two keys is specially designed for social recovery in case that you lost your phone and 2FA devices. 

![](../res/blog/0_q-o2ME7lRtdgCqkC.png)

## Go "passwordless" - You won't lose your passwords if you do not have them

So far most service providers require end users to store and protect their own authentication password. In case of forget or lose, user can still recover from centralized service providers (such as bank, website etc), but there is not possible to get recovered from blockchain since it is decentralized. Shirking the responsibility to end user would be the best solution until now. Gluon takes the responsibility.

## Path to decentralized passwordless

There were passwordless solutiosn before. Most of them are just encrypted secrets (your password) and stored in some centralized location. End users probably do not know where they are stored, but the service provider actually knows, so do hackers. Gluon make all of these information "zero knowledge" to all of us. you do not know, we do not know, the miners do not know, nor do hackers. This seems impossible but we turn it into possible with the help from blockchain and trusted computing technologies. 

## Gluon is one of the TEA projects

TEA (Trusted Execution and Attestation) is a project that builds T-rust framework and applications such as Gluon that running on top. Gluon is an software running on TEA nodes. A TEA node is a HSM(Hardware Secure Module) runs T-rust consensus algorithm by distributed miners. 

![relationship tea](https://cdn-images-1.medium.com/max/1120/1*2O7WDwTwH4DlIr4zbOMxng.png)





