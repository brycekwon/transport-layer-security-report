# Table of Contents

- [1. Overview of Transport Layer Security](#1-overview-of-transport-layer-security)
  - [1.1. Introduction to Transport Layer Security](#11-introduction-to-transport-layer-security)
  - [1.2. Introduction to Cryptography](#12-introduction-to-cryptography)
    - [1.2.1. Symmetric Encryption](#121-symmetric-encryption)
    - [1.2.2. Asymmetric Encryption](#122-asymmetric-encryption)
    - [1.2.3. Diffie-Hellman Key Exchange](#123-diffie-hellman-key-exchange)
- [2. Design of Transport Layer Security](#2-design-of-transport-layer-security)
  - [2.1. Establishing a Reliable Connection](#21-establishing-a-reliable-connection)
  - [2.2. Transport Layer Security Handshake](#22-transport-layer-security-handshake)
    - [2.2.1. Client Hello](#221-client-hello)
    - [2.2.2. Server Hello](#222-server-hello)
    - [2.2.3. Server Certificate](#223-server-certificate)
    - [2.2.4. Server Key Exchange](#224-server-key-exchange)
    - [2.2.5. Server Hello Done](#225-server-hello-done)
    - [2.2.6. Client Key Exchange](#226-client-key-exchange)
    - [2.2.7. Client Change Cipher Spec](#227-client-change-cipher-spec)
    - [2.2.8. Client Finished](#228-client-finished)
    - [2.2.9. Server Change Cipher Spec](#229-server-change-cipher-spec)
    - [2.2.10. Server Finished](#2210-server-finished)
  - [2.3. Transport Layer Security Record](#23-transport-layer-security-record)
- [3. Quantum Resistant Trasnport Layer Security](#3-quantum-resistant-transport-layer-security)
  - [3.1. Introduction](#31-introduction)
  - [3.2. Background Information](#32-background-information)
    - [3.2.1. Quantum Key Distribution](#321-quantum-key-distribution)
    - [3.2.2. Post Quantum Cryptography](#322-post-quantum-cryptography)
    - [3.2.3. Hybrid Transport Layer Security](#323-hybrid-transport-layer-security)
  - [3.3. Results and Discussion](#33-results-and-discussion)
- [4. Conclusion](#4-conclusion)

# 1. Overview of Transport Layer Security

## 1.1. Introduction to Transport Layer Security

Transport Layer Security (TLS) is the successor to Secure Sockets Layer (SSL), a protocol designed to provide a layer of privacy and security between two communicating applications. Its primary function is to ensure that all packets transmitted between a client and server remains end-to-end encrypted, making it theoretically unreadable by external parties. This encryption is crucial in maintaining the integrity and confidentiality of the data being transmitted.

Despite not being a typical part of the layered application stack, TLS is usually added between the application layer and transport layer. This is possible due to the independent nature of these layered applications, which typically do not need to know the contents of their payload, only the source and destination. As a result, TLS and similar protocols are often added as an extension of the preceding protocol, typically denoted with a "Secure (S)" tag at the end of the name. Some popular examples of this upgrade include HTTP becoming HTTPS, SMTP evolving to SMTPS, and FTP turning into FTPS. These secure versions ensure that data transmission over the internet is safe and protected from potential threats.

From a high level, TLS operates similar to sending a sensitive package. Picture yourself with a secret message that you wish to send. To maintain its secrecy during transit, you would place it in a locked box. This box can only be opened with a unique key, which is known solely to you and the intended recipient. In a similar vein, TLS uses a process known as the TLS Handshake to negotiate and agree upon a shared secret key. This key is then used to encrypt and decrypt all subsequent communication.

==Here, we will be explaining TLS from TLSv1.2. Explaining TLS starting from TLSv1.2 before moving on to TLSv1.3 can be beneficial for a few reasons. First, TLSv1.2 is widely used and understood, providing a solid foundation for learning the principles of secure communication. It introduces important concepts such as the handshake protocol, cipher suites, and public key infrastructure. Second, understanding TLSv1.2 can help highlight the improvements made in TLSv1.3. For instance, TLSv1.3 has a simplified handshake process and removes support for insecure or outdated cipher suites, both of which are easier to appreciate with knowledge of TLSv1.2. Lastly, many systems still use TLSv1.2, so understanding it remains practically useful. Therefore, starting with TLSv1.2 can provide a more comprehensive and practical understanding of TLS.

## 1.2. Introduction to Cryptography

Cryptography, while an intricate and expansive topic, is essential to understanding the foundational concepts of how TLS secures data transmission. Although the mathematics behind it are not fundamentally necessary in this aspect, it is crucial to be aware of the two main types of cryptography, their strengths and weaknesses, and the key features they provide.

### 1.2.1. Symmetric Encryption

Symmetric encryption is likely the first type that comes to mind when discussing cryptography. This method involves using a single secret key to both encrypt and decrypt data. Its simple nature makes computations quick and efficient, making it useful for bulk data. However, challenges arise when there is a need to share encrypted data, as this would necessitate the sharing of the secret key. If the key is compromised in any way, which is likely during the sharing process, it would jeopardize the security of the encrypted data.

### 1.2.2. Asymmetric Encryption

On the other hand, asymmetric encryption, also known as public key cryptography (PKC), is considerably more complex. This system employs two keys: a public key, which can be revealed to anyone, and a private key, which must be kept secret. Their mathematical relationship allows for encryption done by the public key to only be decrypted by the private key, and vice versa. This feature facilitates easy data encryption between two parties, as all that is needed is the public key of any participant, who can decrypt the data with their private key. The reversal of this process, where one encrypt data with their private key, allows for a process known as digital signatures, providing a level of authenticity. However, challenges arise when there is a need to encrypt bulk data, as asymmetric encryption is slow and computationally expensive.

### 1.2.3. Diffie-Hellman Key Exchange

A key exchange algorithm is a method that allows two parties to generate a shared secret, leveraging the features of Public Key Cryptography (PKC). This shared secret is crucial for secure communication between the two parties. The algorithm ensures that even if the communication is intercepted, the shared secret remains secure and unknown to the interceptor.

Among the various types of key exchange algorithms, the Diffie-Hellman Key Exchange is the most widely used and favored. This algorithm can be metaphorically compared to the process of mixing paint. It’s easy to mix two colors together, but extremely challenging to separate them back into their original colors. In the context of the Diffie-Hellman Key Exchange, both parties start with a known public value. They then combine this public value with their uniquely generated private key to create a public key. This public key is then exchanged with the other party, who combines it with their private key to create the shared secret. This shared secret can then be used for symmetric encryption.

---
# 2. Design of Transport Layer Security

## 2.1. Establishing a Reliable Connection

The process of incorporating TLS into the layered stack starts with the creation of a reliable connection between client and server. As TLS is not a standalone layer, it is dependent on the assistant other layers for its effective operation. That's where the Transmission Control Protocol (TCP) becomes crucial. TCP is the preferred protocol for TLS because of its ability to resist transport errors, lost packets, and possible interruptions during transmission. Its robustness and reliability make it an ideal choice for ensuring the smooth function of TLS.

In the absence of a reliable connection, the integrity and functionality of data transmission can be severely compromised. Without reliability, data packets may be lost, arrive out of order, or become corrupted during transmission. This can lead to incomplete, incorrect, or nonsensical information being received. In the context of TLS, an unreliable connection could lead to the failure of the secure communication channel, potentially exposing sensitive data to unauthorized parties. Therefore, a reliable connection is crucial for maintaining the security and accuracy of data transmission.

## 2.2. Transport Layer Security Handshake

The TLS handshake is a critical component of the TLS protocol, serving as the initial phase in establishing a secure connection between two parties. This process is vital as it facilitates the authentication of the communicating parties and the establishment of encryption keys before the transmission of the actual data. 

The _TLS Handshake_ primarily involves three key steps, with the entire process completed in just two round-trips between the client and the server.

> 1) Exchange "Hello" messages to establish the TLS version, cipher suite, and random values.
>    
>    a) Client Hello
>    
>    b) Server Hello
> 
> 2) Exchange authentication information to verify the identities of communicating parties.
> 
>    a) Server Certificate
>    
>    b) Client Certificate (optional)
>    
> 3) Derive a session key to use in symmetric encryption once the handshake has been completed.
>    
>    a) Server Key Exchange
>    
>    b) Server Hello Done
>    
>    c) Client Key Exchange
>    
>    d) Client Change Cipher Spec
>    
>    e) Client Finished
>    
>    f) Server Change Cipher Spec
>    
>    g) Server Finished

### 2.2.1. Client Hello

The TLS Handshake commences with a Client Hello message from client to server. This message includes various information to help set the stage for the negotiation process in which client and server agree on the parameters used for the secure connection. The most important tags include supported TLS version, supported cipher suites, and a client-generated random.

```
struct {
	ProtocolVersion client_version;
	Random random;
	CipherSuite cipher_suites;
} ClientHello;
```

### 2.2.2 Server Hello

Upon receipt of the _Client Hello_, the _server_ reciprocates with a _Server Hello_. This response encapsulates the selected values essential for the creation of cryptographic parameters. The most important tags include the chosen TLS version, chosen cipher suite, and a server-generated random.

```
struct {
	ProtocolVersion server_version;
	Random random;
	CipherSuite cipher_suite;
} ServerHello;
```

### 2.2.3. Server Certificate

Before continuing, the server often opts to authenticate itself by sending a Server Certificate message to the client containing a chain of SSL certificates. This allows the client to ensure all communication going forward is being made by the server and has not been tampered by an outside party.

```
struct {
	ASN.1Cert certificate_list
} Certificate;
```

### 2.2.4. Server Key Exchange

Once the cryptographic parameters and authentication have been established, the Key Exchange can begin. The server first defines a public value and combines it with their unique private key to form a public key. This public key along with the public value is sent across the transmission stream to the client.

```
struct {
	select (KeyExchangeAlgorithm) {
		case dh_anon:
			ServerDHParams params;
		case dhe_dss:
		case dhs_rsa:
			ServerDHParams params;
			digitally-signed struct {
				opaque client_random[32];
				opaque server_random[32];
				ServerDHParams params;
			} signed_params;
		case rsa:
		case dh_dss:
		case dh_rsa:
			struct {};
	}
} ServerKeyExchange
```

### 2.2.5. Server Hello Done

Finally, to mark the end of its part of the Key Exchange, the server sends a Server Hello Done message to the client and awaits for a response.

```
struct {} ServerHello;
```

### 2.2.6. Client Key Exchange

Similar to the server's key exchange message, the client combines the provided public value with their private key to derive a public key. It then sends this public key across the transmission stream to the server.

```
struct {
  select (KeyExchangeAlgorithm) {
	  case rsa:
		  EncryptedPreMasterSecret;
	  case dhe_dss:
	  case dhe_rsa:
	  case dh_dss:
	  case dh_rsa:
	  case dh_anon:
		  ClientDiffieHellmanPublic;
  } exchange_keys;
} ClientKeyExchange;
```

At this point, both parties should have all the required cryptographic information to compute a shared pre-master secret by combining their private key with the other party's public key. The final step to deriving the master secret is combining the pre-master secret with the two random values generated in the hello messages.

```
master_secret = PRF(pre_master_secret, "master secret", ClientHello.random + ServerHello.random)
```

### 2.2.7. Client Change Cipher Spec

After generating the master secret, the client informs the server that all subsequent communication will be encrypted using the negotiated cipher suite. This notification is conveyed through a Change Cipher Spec message.

```
struct {
  enum { change_cipher_spec(1), (255) } type;
} ChangeCipherSpec;
```

### 2.2.8. Client Finished

Finally, to complete its side of the TLS Handshake, the client sends a Client Finished message to verify the key exchange and authentication process were successful. This message contains a hash of all previous handshake messages, along with a "client finished" text.

```
struct {
  opaque verify_data[verify_data_length];
} Finished;
```

### 2.2.9. Server Change Cipher Spec

Identical to the Client Change Cipher Spec, the server informs the client that all subsequent communication will be encrypted using the negotiated cipher suite.

```
struct {
  enum { change_cipher_spec(1), (255) } type;
} ChangeCipherSpec;
```

### 2.2.10. Server Finished

Identical to the Client Finished, the server sends a final message to verify the key exchange and authentication process were successful.

```
struct {
  opaque verify_data[verify_data_length];
} Finished;
```

## 2.3. Transport Layer Security Record

The TLS record protocol is responsible for the secure transmission of data between communicating peers. The protocol operates by collecting the data to be transmitted, fragmenting it into manageable blocks, compressing it if necessary, applying a message authentication code (MAC) to ensure the integrity of the data, encrypting with the already derived session keys and transmitting the resulting data to the other peer.

Here, symmetric encryption is used with the derived master secret to encrypt all outgoing communcations and decrypt all incoming communications.

- Dividing outgoing messages into manageable blocks, and reassembling incoming messages. This ensures that data is transmitted efficiently and reliably.
- Compressing outgoing blocks and decompressing incoming blocks (optional). This can help to reduce the amount of data that needs to be transmitted, improving performance.
- Applying a Message Authentication Code (MAC) to outgoing messages, and verifying incoming messages using the MAC. This helps to ensure the integrity of the data and protect against tampering.
- Encrypting outgoing messages and decrypting incoming messages. This provides confidentiality, preventing unauthorized parties from reading the data.

---
# 3. Quantum Resistant Transport Layer Security

The research paper introduces Quantum Key Distribution (QKD) and Post-Quantum Cryptography (PQC) as the first quantum-resistant technologies capable of withstanding quantum computer attacks. These technologies modify the foundation of cryptographic operations to counteract the computational advantages of Shor’s algorithm.

However, no existing works combine PQC and QKD at different stages of the TLS protocol for enhancing its security, nor provide practical measurements of the performance constraints and communication cost of integrating QKD and PQC in TLS.

The authors present a hybrid quantum-resistant TLS protocol enabled by both QKD and PQC, aiming to analyze the performance implications of this solution in an experimental networking scenario. The paper’s contributions include the first experimental demonstration integrating commercially available QKD devices with PQC algorithms into an industry-ready TLS implementation, proposing and analyzing two novel mechanisms for combining PQC-based and QKD-based secret keys, and evaluating the practicality of integrating the proposed hybrid quantum-resistant TLS protocol in an experimental network scenario.

The paper concludes that while the adoption of the proposed quantum-resistant TLS protocol comes at a considerable communication cost, it significantly augments the security of the communications, now protected by two different quantum-resistant cryptographic assumptions.

## 3.1. Introduction

The emergence of quantum computers introduces a significant threat to the security of modern day encryption methods. Current encryption schemes rely on the computational difficulty of solving complex, mathematical problems, which even classical supercomputers struggle to tackle. However, quantum computers, with their unique capabilities, can efficiently break through these problems, rendering current encryption techniques useless.

Symmetric encryption, a widely used method, remains relatively secure against quantum attacks. However, its effective key length is reduced by half, meaning that a 256-bit symmetric key would only provide 128-bit security against quantum attacks. While still considered secure, this may necessitate the use of longer symmetric keys in the future to maintain the same level of protection.

The most pressing concern lies with asymmetric encryption, which relies on the mathematical difficulty of factorization and discrete logarithm problems. These problems are considered computationally intractable for classical computers but can be efficiently solved by quantum computers. This poses a significant challenge to the long-term security of confidential data, especially as quantum computing technology continues to advance.

Recent findings demonstrate that the adoption of a PQC-only approach enhances the TLS handshake performance by approximately 9 % compared to classical methods. Furthermore, our hybrid PQC-QKD quantum-resistant TLS comes at a performance cost of approximately 117 % during the key establishment process. The quantum computing threat has recently raised the need for
current networks to transition towards implementing quantum-resistant
communications. In this regard, there are industry, academic and gov-
ernmental projects that are pushing towards the adoption of both QKD
and PQC within network security protocols such as TLS.

Post-Quantum Cryptography (PQC) and Quantum Key Distribution (QKD) are designed to offer robust security against potential threats posed by quantum computers. PQC ensures security within the random oracle model, while QKD provides an inherently secure environment resistant to exhaustive key search or cryptanalytic attacks, even those originating from quantum computers. By combining QKD and PQC in key exchange, the key remains safe as long as one of the algorithms is secure, addressing the problem of key exchange. Quantum-resistant key exchange is a more urgent issue than quantum-resistant authentication due to the potential for ‘harvest now, decrypt later’ attacks.

The combination of both technologies allows for the alleviation of existing fears towards adopting PQC-based quantum-resistant communications for Authenticated Key Exchange (AKE) in the advent of possible algorithmic breakthroughs. It enables crypto-agile communication systems and considerably augments the security of the Transport Layer Security (TLS) protocol via multiple quantum-resistant cryptographic assumptions. This approach immediately confronts and mitigates the risks associated with potential ‘harvest now, decrypt later’ attacks, addressing a significant security vulnerability in the current landscape of quantum computing.

## 3.2. Background Information

### 3.2.1. Quantum Key Distribution

Quantum Key Distribution (QKD) is a cryptographic technique that aims to securely distill a secret key by transmitting quantum signals between authenticated partners. In a typical QKD scenario, there are two authorized partners, a transmitter and a receiver, connected via two channels. The first is a quantum channel used to transmit quantum signals that are later used to distill a secret key. The second is an authenticated service channel used for key distillation and post-processing, allowing both the transmitter and receiver to perform tasks such as error correction and privacy amplification on the key.

Unlike classical and Post-Quantum Cryptography (PQC) algorithms that rely on the mathematical complexity of solving a given problem, the security of QKD relies on two theorems of quantum physics: the no-cloning theorem and Heisenberg’s uncertainty principle. These principles state that it’s impossible to obtain an identical copy of an arbitrary unknown quantum state and that the measurement of a quantum signal causes a perturbation in its state that cannot be recovered. Therefore, any attempt to eavesdrop on the quantum channel will inevitably alter the state of the quantum signals exchanged, alerting the involved parties of the presence of an eavesdropper. After the transmission of the quantum signals, both the transmitter and receiver can execute a series of post-processing operations to estimate how much information about the quantum signals has been leaked to an outside party. If these operations reveal the presence of an eavesdropper on the quantum channel, the protocol is aborted. Otherwise, a key can be distilled, the security of which is guaranteed by the laws of quantum physics. The resulting shared keys are stored symmetrically in both the transmitter and receiver’s key management.

### 3.2.2. Post-Quantum Cryptography

Post-Quantum Cryptography (PQC) is a collection of cryptographic algorithms designed to remain secure even if an attacker has access to a fault-tolerant quantum computer. Unlike Quantum Key Distribution (QKD), PQC maintains the same functional principles seen in classical asymmetric cryptography, such as key-pair generation, digital signatures, and key encapsulation/decapsulation. While classical cryptography’s mathematical problems have been proven insecure against quantum computers, PQC bases its security on mathematical operations like lattice-based or hash-based cryptography, which quantum algorithms offer little to no speed advantage. PQC algorithms are not theoretically secure against quantum attacks, but they are considered to be much more secure than the algorithms that are currently used.

However, some post-quantum algorithms face the challenge of meeting system usability and scalability demands while remaining safe against quantum computers. The longer key and signature lengths required by different PQC algorithms might lead to performance and scalability issues, potentially necessitating modifications to protocols and infrastructures. As PQC is a novel area in cryptography with ongoing research efforts, there’s a risk that algorithms currently considered quantum-resistant might become vulnerable to future algorithmic breakthroughs.

### 3.2.3. Hybrid Transport Layer Security

With post-quantum encryption algorithms emerging as a promising solution to the issues, they are still very new and have not undergone the rigorous testing as traditional encryption algorithms. To bridge this gap, hybrid TLS has been proposed as a pragmatic approach that combines the strengths of both traditional and post-quantum encryption. By utilizing both types of algorithms, hybrid TLS ensures security even if one type is compromised. This dual-pronged approach safeguards the integrity and confidentiality of data transmitted over TLS connections, offering robust protection against both current and future cryptographic threats.

## 3.3 Results and Discussion

The text discusses the performance of a proposed post-quantum ciphersuite, Dilithium3 and Kyber768, compared to a classical ciphersuite combining RSA-1024 for digital signatures and ECDHE-SECP256 for key establishment. The classical ciphersuite performs slightly faster, taking approximately 99 ms compared to the post-quantum ciphersuite’s 101 ms. However, the classical ciphersuite is not considered secure and is not used in practice. In contrast, the lighter version of the post-quantum solution (Dilithium1 + Kyber512) is the fastest ciphersuite for generating a client key exchange and server key exchange, with a difference of 11 ms over its classical equivalent (RSA1024 + SECP256).

The integration of Quantum Key Distribution (QKD) with Post-Quantum Cryptography (PQC) generates an added overhead of approximately 117% compared to the classical control group (SECP384 + x25519). The overall time of the handshake depends mainly on the overhead added by QKD. Both Kyber512 and Kyber768 (PQC) perform fairly well. The two methods of combining QKD and PQC for the key exchange, either by concatenating both premaster secrets or by XORing two master secrets, have negligible differences in terms of performance (less than 1 ms).

The work introduces a novel scheme for integrating Post-Quantum Cryptography (PQC) and Quantum Key Distribution (QKD) into Transport Layer Security (TLS). This architecture enhances the security of TLS against quantum computing threats and maintains high performance in session key generation. In restricted network conditions, PQC outperforms classical cryptographic algorithms by approximately 10% of the overall handshake time. However, the use of QKD-based keys on top of PQC adds an overhead of approximately 117%, mainly due to the performance of the key retrieval API exposed by the QKD equipment. Despite this, the added overhead is justified as the resultant master secret’s security is guaranteed by two different quantum-resistant cryptographic assumptions.

The work also introduces a quantum-resistant TLS architecture that can combine QKD-based keys on both client and server sides with any other classical or PQC cryptographic algorithm of choice. This paves the way towards combining multiple cryptographic algorithms to achieve maximum security against known and future unknown attacks. However, for this experimental solution to be widely used in real communication networks, several open research questions and challenges require further investigation. These include the integration of the proposed solution into TLS 1.3, the communication of future implementations with network controllers in real-time, the integration of quantum-resistant solutions with Software-Defined Networking (SDN) networks, and the development of ultra-fast key retrieval APIs to reduce the performance cost of QKD.

The paper concludes that while the adoption of the proposed quantum-resistant TLS protocol comes at a considerable communication cost, it significantly augments the security of the communications, now protected by two different quantum-resistant cryptographic assumptions.

---
# 4. Conclusion

Transport Layer Security is a vital protocol that safeguards the confidentiality and integrity of data exchanged between web servers and clients. It is instrumental in protecting sensitive information such as financial data and personal details, making it a cornerstone of modern online communication. Without TLS, online interactions would be susceptible to various security threats, including eavesdropping and data tampering.

However, the advent of quantum computing poses a significant challenge to traditional cryptography, including TLS. Quantum algorithms could potentially break the mathematical foundations of current encryption schemes, rendering them vulnerable. In response to this threat, researchers are developing post-quantum cryptography (PQC) algorithms that are resistant to quantum computing attacks. These algorithms, based on different mathematical principles than traditional cryptography, are more resilient to quantum-based cryptanalysis. The integration of PQC algorithms into TLS is an ongoing effort, with standardization bodies working to define standards for quantum-resistant TLS protocols. As quantum computing technology continues to mature, transitioning to quantum-resistant TLS will be crucial to maintaining the security of our online communications.
