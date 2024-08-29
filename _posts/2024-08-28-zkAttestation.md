---
layout: post
title: Attesttation
# subtitle: Excerpt from Soulshaping by Jeff Brown
# cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [MPC-TLS, ZK-TLS]
author: Xiang Xie
usemathjax: true
---
# Preserving Reality: The Crucial Role of Attestation in Anti-FakeAI.
**TL;DR**: As artificial intelligence (AI) continues to advance, its ability to simulate the real world becomes incredibly powerful. However, this progress also brings about the rise of FakeAI, which has the potential to blur the lines between simulated and real-world content, leading to confusion for individuals. Cryptography emerges as the primary defense against this threat, with attestation serving as a crucial mechanism to ensure content authenticity and validate human involvement. This article provides an in-depth exploration of attestation, including its definitions, challenges, and proposed solutions.

## 1. Why Attesttation: Safeguarding Against AI Simulations
As AI continues to evolve, its ability to simulate the real world with unprecedented accuracy raises profound concerns. While AI's capabilities hold promise for various applications, they also introduce the risk of blurring the lines between the real and simulated worlds. In this era of FakeAI, where synthetic content can appear indistinguishable from reality, the importance of attestation emerges as a critical safeguard against potential deception and manipulation.

Attestation revolves around verifying the authenticity and integrity of information, particularly concerning its source. It acts as a mechanism to validate the origin and trustworthiness of data. In the context of combating AI simulations, attestation plays a pivotal role in preserving the distinction between the real and simulated worlds.

Attestations can serve as a deterrent against malicious actors seeking to exploit AI simulations for deceptive purposes. By requiring rigorous validation of data sources through attestations, organizations and individuals can mitigate the risks associated with fake or manipulated content. This proactive approach helps safeguard against misinformation, fraud, and other harmful consequences stemming from AI simulations.

## 2. What is Attestation
Upon encountering attestation for the first time, people often wonder, "What exactly is attestation?" The distinctions among attestor, prover, attestation and proof can be a source of confusion for many.

**Proof**, refers to a mathematical demonstration or verification of the validity or correctness of a statement or claim.

**Attestation**, refers to the process of verifying the authenticity or integrity of information, **typically involving the verification of the source or origin of data, software, or hardware.** Attestation ensures that the entity providing the information is genuine and has not been tampered with or compromised.

**Attestation revolves around verifying the source of data, typically tied to an attester that furnishes the attestation. Conversely, proof encompasses a broader concept, centering on general statements and claims. It's essential to recognize that asserting the source of data is itself a claim.**


**An example of proof.** In the blockchain, a simple example of a proof is when a user, owning the matching private key of an address, signs a transaction. This signed transaction serves as a "proof" to demonstrate ownership and the ability to spend the assets associated with that address.

**An example of attestation.** In the context of attestation, consider proving your education to a university. By achieving a satisfactory GPA and completing the required courses, you earn an attestation, essentially an education certification issued by the university. Later, when you seek employment at a company (acting as the verifier), your attestation becomes a valuable proof of your education level, supporting your qualifications for the job.

## 2. Challenges

Certainly, attestation emerges as a crucial mechanism for authenticating internet data, but it still encounters a few challenges.

#### Data Authenticity
One of the challenges revolves around ensuring the authenticity of the data. This necessitates a means to verify that the data indeed originates from the source claimed by the user. The complexity lies in the difficulty of engaging Web2 data providers (such as Banks, Google, Facebook, X, etc.) in collaborative efforts with attestors. This can result in a substantial cost of business development and negotiation.

**Ideally, an optimal solution would involve a "clever" approach that ensures data authenticity without any modifications to the existing data providers.** This process should be seamlessly performed between users and attestors, all while keeping data providers unaware of the proceedings. This method aligns with the principle of empowering users to regain control over their data, allowing them to retrieve and utilize their information autonomously.


Indeed, certain types of data inherently offer a mechanism for verifying the authenticity of data providers. This specific data is signed by the providers in accordance with established internet standard protocols. In such cases users can directly generate a proof, as a self-attestation, that can be verified by the Web3 ecosystem such as smart contracts. However, it's crucial to acknowledge that a significant portion of Web2 data lacks this inherent property, necessitating alternative approaches such as attestations to ensure the authenticity and integrity of the information.

#### Privacy
Ensuring privacy is an ever-pressing concern in the realm of data management. Ideally, users should not be compelled to disclose sensitive information to attestors. The ultimate objective of attestation in authenticating internet data is to minimize the trust of attestors. This becomes particularly vital when incorporating valuable Web2 data into Web3 applications, especially within the transparency of smart contracts. Achieving this necessitates a privacy-preserving approach to attestation and handling private data.

## 3. Technique Stacks for Attestations
We provide a brief overview of existing techniques for attestations as follows, highlighting both their strengths and limitations, along with the underlying trust assumptions required. Further exploration of these techniques is conducted in subsequent sections.
<p align="center">
    <img src = "https://hackmd.io/_uploads/r1iy4Ugh6.png" style="zoom:55%"/>
</p>


### 3.1 Oracle
Oracles, such as Chainlink, play a pivotal role in the Web3 ecosystem, especially within the domain of DeFi. At the forefront of their contribution is the provision of essential data, particularly token prices. Undoubtedly, this data serves as the cornerstone for a multitude of financial applications currently thriving in the web3.

The fundamental mechanism of current Oracles relies on consensus protocols to reach agreement on specific data values. While this approach effectively addresses challenges related to data authenticity, it primarily caters to public data. The reason is that achieving consensus on data necessitates exposing it to all validators, presenting a potential conflict with data privacy principles. This is because, before achieving consensus, the validators have to see the data in plain. This aspect likely explains the predominant focus on public token prices, where the need for consensus aligns with the inherently public nature of such financial data.

### 3.2 ZKP for Signed Data

Several standard internet protocols empower users to engage in self-attestation, with two of the most popular protocols being associated with Email and Single Sign-On (SSO) systems.

#### DKIM (Domain Keys Identified Mail) 
DKIM is commonly used for email verification. A large amount of Web2 data flows through email, and almost all emails are signed by the sending domain by using DKIM in the following format.
```=
rsa_sign(sha256(from:<>, to:<>, subject:<>, <body hash>,...), private key)
```

With this signature, a user can prove the receipt of an email from a specific sender, say X, by sharing the email header from name@X.com to you@domain.com. Anyone can fetch the public key of X and use it to verify the signature.  Since the email header contains a hash of the email body, it becomes possible to prove the content of the email by sharing the email body. The public key holder can then verify the content.

To enhance security and reduce the cost of on-chain verification. The user can then apply zero-knowledge proofs to prove the authenticity of the email and content inside.

The trust assumptions inherent in this protocol primarily rely on the trustworthiness of the sender's mail server and the secure means of fetching a public key. These assumptions are generally considered acceptable. The [ZK Email](https://blog.aayushg.com/zkemail/#fn:1) project is dedicated to the development of this protocol. Numerous other projects have also leveraged this protocol to build interesting applications.


#### OpenID Connect
OpenID Connect is an authentication protocol allowing users to log into different websites using the same identity provider. For example, there are SSO options where you can use your Google or Facebook account to sign in to various sites across the web, without needing to create new usernames and passwords for each one. 

In this scenario, when users employ OpenID Connect for SSO, the OpenID provider (e.g., Google) issues a signed JSON Web Token (JWT), described below, containing user identity information to the third-party web servers. The third-party web server then validates the signed JWT by fetching the public key of the OpenID provider and checking the signature.

```Jason=
{
"sub": "1234567890", # User ID
"iss": "google.com", # Issuer ID
"aud": "4074087", # Client or App ID
"iat": 1676415809, # Issuance time
"exp": 1676419409, # Expiry time
"name": "Alice",
"email": "alice@gmail.com",
"nonce": "7-VU9fuWeWtgDLHmVJ2UtRrine8"
}
```

It is somehow natural to combine zero-knowledge proofs with JWT to enable the utilization of existing identities for accessing Web3 applications. The [zkLogin paper](https://arxiv.org/pdf/2401.11735.pdf) proposes a nice way to bind the blockchain account and JWT together by replacing the `nonce` entry with a public key, which can be used to authenticate transactions. We refer to the paper for more details.


The trust assumptions are similar to the DKIM case. They rely on the trust of the OpenID provider and the secure means of fetching a public key.


### 3.3 Integrating TLS for General Web Data

As previously discussed, while DKIM and OpenID Connect are extensively deployed in Web2, their scope is limited to a specific subset of Web2 data. To delve deeper into the possibilities and diversity of Web2 data, it becomes essential to consider general web data.

This naturally brings our focus to the standard Transport Layer Security (TLS) protocol, it is nearly ubiquitous in facilitating the transfer of Web2 data. For instance, every URL that begins with `https` utilizes the TLS protocol.


TLS is a standard internet protocol designed to ensure both authenticity and confidentiality in communication. The TLS protocol involves two key roles: a TLS Client and a TLS Server. In a TLS-enabled connection, the client can verify that the server it is communicating with is the intended one. Additionally, all messages exchanged between the client and server are encrypted using a random key known only to these two parties.
<p align="center">
    <img src = "https://hackmd.io/_uploads/r1kKQ1a5T.png" style="zoom:55%"/>
</p>
 

The TLS protocol operates in two main phases: the Handshake and Record phases. During the Handshake phase, the TLS Client verifies the Server's identity, and both parties establish a shared session key $K$ through key-exchange protocols. In the subsequent Record phase, all communication between the Client and Server is encrypted using the shared session key $K$.

#### Trusted Execution Environment (TEE)
[Town Crier](https://eprint.iacr.org/2016/168.pdf) is notable as the first system to authenticate TLS data via Trusted Execution Environments (TEEs). The core concept involves the end user delegating the role of the TLS Client to the TEE, which is operated by the attestor. Essentially, the real TLS Client operates within the enclave of the TEE. The TEE then signs the response data received from the TLS Server, creating an attestation that authenticates the interaction. The Relayer assists in efficiently relaying HTTPS or TLS messages, minimizing overhead within the TEE.
<div align=center>
    <img src = "https://hackmd.io/_uploads/SkSLAJa5p.png" style="zoom:55%">
    
</div>

The trust assumption in this solution is straightforward: trust the TLS Server and TEE. This ensures the guarantee of authenticity and privacy, with the assumption that the attestor running the TEE cannot access the data -- a natural expectation given the inherent definition of TEE, which designed to provide a secure and isolated environment.

#### MPC-TLS

An alternative method for authenticating TLS payload involves leveraging multi-party computation (MPC), specifically in a two-party computation scenario. Pioneered by [DECO](https://arxiv.org/abs/1909.00938), this approach entails the user and attestor jointly emulating a TLS Client using MPC. During the protocol, random key shares, denoted as $K_1$ and $K_2$, are generated for each party. These shares collectively represent the session key $K$. The attestor will sign the encrypted messages received from the TLS Server as an attestation.

<div align=center>
    <img src = "https://hackmd.io/_uploads/S1bELx656.png" style="zoom:55%">
    
</div>

The underlying principle of this method lies in preventing a malicious user from independently controlling the session key. In this approach, both the user and attestor actively participate in "watching" the message channel between the TLS Client and Server concurrently. As a result, the attestor can ensure the authenticity of the encrypted data (note that the attestor alone can not decrypt the message), which is guaranteed by the TLS protocol itself.

However, the original DECO protocol is too heavy to be practical. The main issue arises from the utilization of a general-purpose maliciously secure two-party computation (2PC) protocol and zkSNARKs to prove cipher operations like AES/Chacha. The computation time, communication size, and RAM consumption associated with these processes are substantial, making it impractical for execution on the client side.

[TLSNotary](https://tlsnotary.org/), as a subsequent effort, has proposed optimizations to enhance the DECO protocol and have actively contributed to an open-source library over the years. A recent [paper](https://eprint.iacr.org/2023/964.pdf) from [PADO Labs](https://padolabs.org/) introduces an innovative technique named garble-then-prove, demonstrating a remarkable performance improvement by an order of magnitude. This method relies on VOLE-based interactive zero-knowledge proofs (IZK), leveraging the cutting-edge IZK technology developed by their academic team.

The trust assumption in this type of solution lies on the trust of TLS Server and attestor. It's essential note that the attestor can not get any information of the private data, but can still ensure the authenticity of the data.


#### ZK-TLS (Attestor-in-the-middle)
As described in the DECO paper, there is an interesting solution designed to circumvent the need for the MPC protocol between the user and the attestor. This alternative approach involves situating the attestor in the middle of the user and the TLS Server, functioning as a message relayer. 

The attestor directly acquires the encrypted messages exchanged between the TLS Server and the Client. Subsequently, the user utilizes a zero-knowledge proof to demonstrate their knowledge of the underlying session key and the private messages encapsulated within the transmitted messages. The attestor will sign on the ciphertext if all the checks pass. This solution is applied by the [reclaim protocol](https://www.reclaimprotocol.org/).

<div align=center>
    <img src = "https://hackmd.io/_uploads/Hkyodlpca.png" style="zoom:55%">
    
</div>

This method yields performance gains. However, as pointed out in the DECO paper, it introduces other network assumptions. For data integrity, it assumes that the attestor can reliably connect to the TLS Server throughout the session. Moreover, it assumes that messages sent between the attestor and the TLS Server cannot be tampered with by the user, who knows the session keys and could thus modify the session content. 

The trust assumption remains consistent with the MPC-TLS solutions, incorporating additional network assumptions. Trust is required in both the TLS Server and the attestor.

### 3.4 Attestation Frameworks
To further minimize reliance on attestors and promote decentralization, various attestation frameworks have been introduced, such as [EAS](https://attest.sh/) and [Verax](https://www.ver.ax/). These frameworks provide generic methods, allowing anyone to register as an attestor within the network, regardless of the attestation method used. This approach offers flexibility for users and applications to choose any attestor and attestation method based on their specific requirements.

#### Ethereum Attestation Service (EAS) 
EAS is an open-source infrastructure public good for making attestations onchain or offchain. It is a standard layer where any entity can make attestations about anything. This primitive and ledger of attestations will help to decentralize more than just money and assets. It enables to build reputation systems, voting systems, governance systems, decentralized social media, provenance of goods, knowledge and social graphs, and more. EAS runs on two smart contracts: one for registering attestation Schemas and another for attesting with them. Schemas can be registered for any use case.

#### Verax
Verax, mainly developed by the Linea team, is a shared, public attestation registry that can deployed to EVM chains. It can be used by dApps to store attestations that is of public interest, that can be easily accessed and composed together by anyone that's interested. It is designed to be deployed as a single instance per network, so that all dApps on that network can issue their attestations to the same place, so that they can be easily discovered and consumed. 


## 4. Conclusion
This article covers the definition, necessity, challenges, and potential solutions of attestation, highlighting their trust assumptions. It is important to point out that attestation is only the beginning, the value of Internet data lies on the computation!

We thank Feng Liu for the thorough review of the initial version of the article
