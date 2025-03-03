---
layout: post
title: zkTLS Benchmark
# subtitle: Excerpt from Soulshaping by Jeff Brown
# cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [zkTLS, Benchmark]
author: Xiang Xie, Xiao Wang
usemathjax: true
---
# Unearthing the Reality of zkTLS: A Benchmarking and Cryptanalysis Report
**TL;DR** This article provides an open-source benchmark framework for existing open-source zkTLS libraries across various platforms and network conditions. It finds that the [garble-then-prove](https://eprint.iacr.org/2023/964) system proposed by [Primus](https://primuslabs.xyz/) is up to an order of magnitude faster than other MPC-TLS solutions. Additionally, Primus’s [QuickSilver](https://eprint.iacr.org/2021/076)-based Proxy-TLS protocol is up to 30x faster than alternatives. We welcome PR from other teams to join the [open-source benchmark](https://github.com/primus-labs/zktls-bench/tree/main) effort. Additionally, we also point out a security flaw in some Proxy-TLS implementations, which might allow malicious clients to prove false statements in certain cases. We have conducted the necessary responsible disclosure to relevant teams and want to clarify that we have not performed an actual attack on their code.

zkTLS, also known as web proofs, is a cryptographic protocol that ensures the authenticity and privacy of the data transferred over the Transport Layer Security (TLS) protocol, such as in all HTTPS-based web applications. zkTLS opens up possibilities for bringing Web2 data to Web3 with verifiability, while preserving privacy.

$s_i$
## An Overview of TLS
Before diving into the detailed protocol of zkTLS, it’s useful to first review the high-level flow of TLS, as this will help us understand the differences of zkTLS approaches.

The TLS protocol involves two parties: the Client and the Data Source (which we'll refer to as the Server for consistency with zkTLS). TLS operates in two main phases: the Handshake phase and the Record phase.

### Handshake Phase
In the Handshake phase, the Client and the Data Source negotiate a shared secret key known as the pre-master secret (`pms`). This pre-master secret is then used to generate session keys that will be used to encrypt the data during communication.

To generate these session keys, the Key Derivation Function (KDF) is employed. The KDF takes the pre-master secret (`pms`) and other public information (such as the client and server random values) to derive the session keys. This process is typically accomplished using a series of HMAC functions.

Formally, the session keys can be derived as follows:

$\mathsf{keys}\leftarrow \mathsf{KDF}(\mathsf{pms},\mathsf{public_info})$

We refer the details to [RFC 5246](https://datatracker.ietf.org/doc/html/rfc5246) and [RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446).

### Record Phase
In the Record phase, the Client and Data Source use session keys to encrypt and decrypt messages. Most zkTLS protocols use AES-GCM for data transfer. Below is a high-level overview of AES-GCM.

Given a key $\mathsf{K}$ and messages $\mathsf{(m_0,...m_\ell)}$, where each $m_i$ is 16 bytes (assuming AES-128 is used), the ciphertext is of the form

$$\mathsf{AES(K,ctr_0)}\oplus m_0~,...,~\mathsf{AES(K,ctr_\ell)}\oplus m_\ell~,~ \tau$$

Here $\mathsf{ctr}_i$ are counters and $\tau$ is a tag computed from the previous ciphertexts and the $\mathsf{K}$. When decrypting the ciphertext with the key $\mathsf{K}$, the system first checks $\tau$ and then decrypts to recover $\mathsf{(m_0,...,m_\ell)}$. We refer the details to [RFC 5288](https://datatracker.ietf.org/doc/html/rfc5288).

## Two Main Approaches of zkTLS: MPC-TLS and Proxy-TLS
In zkTLS protocols, a third party, known as the **attestor** or **notary**, is introduced to authenticate the data the Client receives from the Data Source, while ensuring that no private information (e.g., passwords) is leaked to the attestor.

The two main approaches to zkTLS in the industry are **MPC-TLS** and **Proxy-TLS**, which are the primary focus of this article.

### MPC-TLS 
In the MPC-TLS approach, the attestor and client run a two-party computation (2PC) protocol to simulate the client side of the TLS handshake. This means the client does not receive the entire session key after the Handshake Phase. Only after the attestor receives the response ciphertexts does the attestor send the secret key shares to the client, enabling the client to decrypt all the ciphertexts.

The high-level flow of MPC-TLS works as follows.
1. The Client and Attestor run 2PC protocols in the Handshake phase. At the end of this step, the Client and Attestor hold key shares of the session keys. The main part of this step is running a 2PC protocol for the KDF function.
2. The Client and Attestor run another 2PC protocol to compute the ciphertext of the request, which includes computing the AES function and the tag.
3. The Client receives the response ciphertext from the Data Source and forwards it to the Attestor.
4. The Attestor sends the key shares to the Client, who, now in possession of the full session keys, proves to the Attestor that the ciphertext is valid and satisfies other properties of the underlying messages.

Note that the Client and Attestor do not run a 2PC protocol to decrypt the response ciphertext.

![image](https://hackmd.io/_uploads/Sky2f5_Yyl.png)

The [DECO](https://arxiv.org/pdf/1909.00938) solution uses authenticated garbled circuits and zkSNARKs in the MPC-TLS approach. However, this method is resource-intensive, both in terms of communication size and computation time, making it difficult to deploy in real-world products.

[TLS Notary](https://tlsnotary.org/) optimizes this approach by using semi-honest garbled circuits and garbled-circuit-based interactive zero-knowledge proofs. While this reduces some of the overhead, the cost remains large.

[Primus](https://primuslabs.xyz/) introduces the [garble-then-prove](https://eprint.iacr.org/2023/964) protocol within the MPC-TLS setting, combining semi-honest garbled circuits with the highly efficient interactive zero-knowledge proof system [QuickSilver](https://eprint.iacr.org/2021/076), along with other optimizations. This results in a solution that is both computationally and communicationally efficient.


### Proxy-TLS

In the Proxy-TLS approach, the Attestor acts as a proxy between the Client and Data Source, forwarding all TLS transcripts. The Attestor can record both the Handshake transcripts and the ciphertexts exchanged between the Client and Data Source. At the end of the protocol, the Client will prove to the Attestor in zero-knowledge about the validity of the ciphertexts.

The motivation behind the Proxy-TLS approach is to eliminate the 2PC protocols used in MPC-TLS, which are the most computationally intensive part. However, this introduces a stronger network assumption: the Attestor must ensure a direct connection to the real Data Source. If this condition is not met, a malicious Client could sit between the Attestor and the Data Source, allowing it to falsify data and prove any claims. While many consider this assumption reasonable, it is still important to highlight.

Depends on statements the Client proves to the Attestor, there are two options adopted by different projects.

#### Option 1: Proving KDF and AES.
In this case, the Client proves the Key in the AES encryption is derived from the `pms` and the message is encrypted with the same key under the ciphertext recorded by Attestor. See [this paper](https://eprint.iacr.org/2024/447.pdf) for more details. 

#### Option 2: Proving public padding and AES.
This option works for HTTPS, where there is a fixed public padding in the response, such as the 200 OK status returned by the Data Source. The Client proves that some of the ciphertexts encrypt the public padding, while others encrypt the message, all using the same key. More details are given [here](https://eprint.iacr.org/2024/733.pdf).

Both options are designed to address the non-committing issue of the AES-GCM cipher suite. In addition, it is crucial to prove that the AES key is consistent across different blocks and matches the output of the KDF function. We have found that some implementations fail to prove this property, allowing a malicious Client to make false claims. We will discuss this in more detail in the next section.

![image](https://hackmd.io/_uploads/BJuTzquYJx.png)

[Primus](https://primuslabs.xyz/) and [Pluto](https://pluto.xyz/) leverage Option 1 in their Proxy-TLS approach, with Primus using QuickSilver and Pluto utilizing Incrementally Verifiable Computation (IVC) systems.

On the other hand, [Reclaim](https://www.reclaimprotocol.org/) adopts Option 2, using Groth16 as the underlying proof system.

## Performance
We benchmark existing open-source zkTLS core libraries, including [Primus](https://primuslabs.xyz/), [TLS Notary](https://tlsnotary.org/), [Reclaim](https://www.reclaimprotocol.org/), and [Pluto](https://pluto.xyz/) (Origo). To conduct the benchmarks, we built an open-source framework that tests performance across different platforms, including x86, ARM, and Browser (WASM). The framework also allows for customization of various network environments. We refer to this Github [repository](https://github.com/primus-labs/zktls-bench/tree/main).

To simulate real-world scenarios—since most applications access websites via browsers, which require sending cookies over TLS—we set the request size to either 1KB or 2KB. The response size is varied from 16 Bytes to 2KB. It’s important to note that actual performance may vary depending on network and platform configurations.

We provide performance metrics for several representative environments.
- Bandwidth : 200 Mbps
- Latency : 10 ms
- Request size: 2 KBytes
- OS: Ubuntu-22.04
- CPU: 8 vCPUs, 3.10GHz
- Memory: 32GB

### MPC-TLS
#### Lib: Primus otls （TLS 1.2）

| Response Size (Bytes) | Uploaded Size (KBytes) | Downloaded Size (KBytes) | Native(x86) Runtime (s) | Native(arm) Runtime (s) | Browser Runtime (s)|Memory (MBytes)|
| ----------------------| ----------------------- | ------------------------ | ----------- | ---------------- |---|-----|
| 16                   | 34,888                     | 1,141                     | 2.56        |2.64| 5.05 |146             |
| 256                   | 34,901                     | 1,141                     | 2.57        |2.62|5.09 |146             |
| 1024                   | 34,930                    | 1,141                     | 2.57        |2.64|5.04 |146            |
| 2048                   | 34,971                     | 1,141                    | 2.58        |2.65|5.11 |146             |


#### Lib: TLS Notary （TLS 1.2）

| Response Size (Bytes) | Uploaded Size (KBytes) | Downloaded Size (KBytes) | Native(x86) Runtime (s) | Native(arm) Runtime (s) |Browser Runtime (s) |Memory (MBytes)|
| ----------------------| ----------------------- | ------------------------ | ----------- |---|--| ---------------- |
| 16                   | 61,089                     | 62,144                     | 20        |32|47| 147             |
| 256                   | 61,121                     | 65,309                     | 20       |33|48| 148             |
| 1024                   | 61,222                     | 75,437                     | 21       |35|49| 148            |
| 2048                   | 61,356                     | 88,940                     | 23       |39|53| 148             |



1. Both Primus and TLS Notary have implemented MPC-TLS protocols for TLS 1.2, and both are memory-efficient across different platforms. In both implementations, the browser runtime is approximately **2x** slower than native executions.
2. Under these metrics, Primus is approximately **10x** faster than TLS Notary in total runtime across different platforms. Primus maintains a stable runtime regardless of response size, whereas TLS Notary experiences an increase in both download size and runtime as the response size grows—though the variation remains relatively small.
3. The primary reason for the benchmark results above is likely Primus’s use of QuickSilver, optimized circuits, and an efficient Garbled Circuit (GC) implementation. **Notably, the TLS Notary team is also integrating QuickSilver with support from the Primus team**. Rough estimates suggest that adopting QuickSilver will significantly reduce download size and lower the computation overhead from approximately 3GC (running GC three times) to 2GC.

### PROXY-TLS
In Proxy-TLS solutions, to simplify benchmarking and ensure fairness as much as possible, we set the request size to zero. This means that none of the implementations prove the request data, allowing the focus to remain solely on the size of the response data.

#### Lib: Primus otls （TLS 1.2）

| Response Size (Bytes) | Uploaded Size (KBytes) | Downloaded Size (KBytes) | Native(x86) Runtime (s) | Native(arm) Runtime (s) | Browser Runtime (s)|Memory (MBytes)|
| ----------------------| ----------------------- | ------------------------ | ----------- | ---------------- |---|-----|
| 16                   | 375                     | 788                     | 0.45        |0.48| 2.31 |71             |
| 256                   | 383                     | 788                     | 0.46        |0.47|2.34 |75             |
| 1024                   | 416                    | 788                     | 0.47        |0.49|2.48 |91            |
| 2048                   | 461                     | 788                    | 0.48        |0.52|2.66 |111             |


#### Lib: Reclaim （TLS 1.2/1.3）

| Response Size (Bytes) | Uploaded Size (1.2/1.3 KBytes) | Downloaded Size (1.2/1.3 KBytes) | Native(x86) Runtime (1.2/1.3 s) |Native(arm) Runtime (1.2/1.3 s)  |Browser  Runtime (1.2/1.3 s) |Memory (MBytes)|
| ----------------------| ----------------------- | ------------------------ | ----------- |---|--| ---------------- |
| 16                   | 32/23                     | 31/22                    | 4.09/3.62        |8.79/8.16|13.88/13.05| 356             |
| 256                   | 34/24                     | 32/23                     | 5.09/4.76       |11.06/10.64|18.76/21.43| 313             |
| 1024                   | 38/29                     | 38/29                     | 8.53/8.43       |18.71/19.10|36.63/49.21| 298            |
| 2048                   | 44/35                     | 44/36                     | 12.92/13.38       |28.61/29.83|60.22/86.21| 306             |

#### Lib: Pluto origo （TLS 1.3）

| Response Size (Bytes) | Uploaded Size (KBytes) | Downloaded Size (KBytes) | Native(x86) Runtime (s) | Native(arm) Runtime (s) | Browser Runtime (s)|Memory (MBytes)|
| ----------------------| ----------------------- | ------------------------ | ----------- | ---------------- |---|-----|
| 16                   | 62                     | 14                     | 41        |56| 193 |1,395             |
| 256                   | 62                     | 14                     | 41        |57|206 |1,395             |
| 1024                   | 63                    | 15                     | 58        |89|293 |1,391            |
| 2048                   | 65                     | 16                    | 76        |119|390 |1,412             |



1. Primus implements Option 1 in Proxy-TLS for TLS 1.2, while Pluto adopts Option 1 for TLS 1.3 using the ChaCha20-Poly1305 cipher suite, primarily because ChaCha20 is more compatible with their IVC system. Reclaim, on the other hand, implements Option 2 for both TLS 1.2 and TLS 1.3. 
2. The total communication size of Reclaim and Pluto is nearly identical, with both being up to **18x** smaller than Primus—as expected given their use of zkSNARKs. 
3. The total runtime of Primus is **5x** to **30x** faster than Reclaim across different platforms. Additionally, Primus consumes **3x** to **5x** less memory, though both remain lightweight and sufficient for most applications. Pluto, however, is significantly slower than both Primus and Reclaim—over **90×** slower than Primus—with substantially higher memory consumption. This is due to its use of IVC, which is highly computationally intensive.
4. The primary reason for the benchmark results above is likely Primus's use of QuickSilver, while Reclaim relies on Groth16 and Pluto utilizes Nova, both of which are computationally intensive. This also explains why Primus's runtime remains relatively stable regardless of response size, whereas Reclaim and Pluto's runtime are highly sensitive to response length. **Note that, as some repositories are continuously updated, performance metrics may vary over time.**

#### Secuirty Issues
When benchmarking Reclaim's implementation, we discovered something counterintuitive. Groth16 is a circuit-dependent proof system, meaning it requires a separate trusted setup for each unique circuit. In the Proxy-TLS setting, the circuit depends on the response length, i.e., the number of AES blocks. However, we found that in Reclaim’s system, users do not need to rerun the setup each time the response length varies, which contradicts the expected behavior.

After confirming with the team, we found that Reclaim actually proves each AES block separately without ensuring that the encryption key remains consistent across all blocks. As a result, a single setup procedure for one AES block is sufficient. However, this implementation may allow malicious Clients to make false claims. Below, we provide a simple explanation of this potential issue. That said, we want to emphasize that we have not conducted an actual attack on their code but are merely pointing out this possible security flaw.

#### Possible Attacks
Suppose the attestor/notary gets two AES ciphertexts as follows.
$$c_0 = \mathsf{AES(K,ctr_0)}\oplus m_0, c_1 = \mathsf{AES(K,ctr_1)}\oplus m$$
where $m_0$ is some public padding information, and $m$ is the data the Client wants to prove.

In Reclaim's implementation, the Client generates two separate Groth16 proofs for $c_0$ and $c_1$ without ensuring that the encryption key $\mathsf{K}$ remains consistent between these two ciphertexts. This allow the malicous Client can conduct the following attack.

The Client chooses some $\mathsf{K}^\prime$, and rewrite $c_1$ as 
$$c_1 = \mathsf{AES(K',ctr_1)} \oplus \mathsf{AES(K',ctr_1)} \oplus \mathsf{AES(K,ctr_1)}\oplus m$$

Therefore, $c_1$ is a ciphertext of $m' = \mathsf{AES(K',ctr_1)} \oplus \mathsf{AES(K,ctr_1)}\oplus m$ under the key $\mathsf{K}'$ and $\mathsf{ctr_1}$. Then the Client can claim the message from the data source is $m'$.

In the case where $m$ is public data, as the Reclaim team explained, the Attestor performs a format check on the message—for example, ensuring that $m$ consists only of numeric characters. Since a randomly chosen $m'$ is highly unlikely to be a valid numeric string, the Attestor would ultimately reject the proof.

However, if $m$ represents private data—such as a bank account balance—and the goal is to prove a statement about it (e.g., the balance is greater than 100), potential attacks may arise. This is because the Attestor cannot access $m$ directly and therefore cannot verify its format. In such cases, a maliciously chosen $m'$ could represent a very large number, leading to an invalid proof being accepted.

## Conclusion
In summary, we found that in the context of zkTLS, zero-knowledge proof is crucial for 1) ensuring the honest behavior of the parties and 2) proving the content in TLS payload. However, we observed an interactive protocol like QuickSilver always outputs alternative solutions due to its scalability.
Based on the benchmark results, which are openly available for extension and verification, we believe interactive ZK like QuickSilver is currently the most suitable approach zkTLS, especially when we push the technology over the browsers.