---
stand_alone: true
category: std
submissionType: IETF
ipr: trust200902
lang: en

title: Cryptographic Asset Discovery and Inventory
abbrev: CADI
docname: draft-liu-cadi-latest
obsoletes:
updates:
date:

# area:
# workgroup:

kw:
  - CADI
  - PQC

author:
 -
  ins: C. P. Liu
  name: Chunchi Peter Liu
  organization: Huawei
  email: liuchunchi@huawei.com
  country: China

normative:
  RFC9849:
  RFC9347:

informative:
  G7:
    title: Advancing a Coordinated Roadmap for the Transition to Post-Quantum Cryptography in the Financial Sector
    date: 2026-01
    target: https://home.treasury.gov/system/files/136/G7-CEG-Quantum-Roadmap.pdf
  CISAACDI:
    title: Strategy for Migrating to Automated Post-Quantum Cryptography Discovery and Inventory Tools
    date: 2024-08
    target: https://www.cisa.gov/sites/default/files/2024-09/Strategy-for-Migrating-to-Automated-PQC-Discovery-and-Inventory-Tools.pdf
  NCCOEFAQ:
    title: Frequently Asked Questions about Post-Quantum Cryptography
    date: 2026-06
    target: https://pages.nist.gov/nccoe-migration-post-quantum-cryptography/FAQ/index.html
  OMBM2302:
    title: MEMORANDUM FOR THE HEADS OF EXECUTIVE DEPARTMENTS AND AGENCIES, Migrating to Post-Quantum Cryptography
    date: 2022-11
    target: https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memo-on-Migrating-to-Post-Quantum-Cryptography.pdf
  NIST-SP-1800-38B:
    title: Migration to Post-Quantum Cryptography Quantum Readiness- Cryptographic Discovery
    date: 2023-12
    target: https://www.nccoe.nist.gov/sites/default/files/2023-12/pqc-migration-nist-sp-1800-38b-preliminary-draft.pdf
  MERCURY:
    title: Mercury - network metadata capture and analysis.
    target: https://github.com/cisco/mercury/
  IBMCBOM:
    title: CBOM
    target: https://github.com/IBM/CBOM
  CDXGEN:
    title: CycloneDX Generator
    target: https://github.com/cdxgen/cdxgen

--- abstract

This document compiles existing Cryptographic Asset Discovery and Inventory (CADI) methods and analyze potential gaps.

--- middle

# Introduction {#intro}

Cryptographic Asset Discovery and Inventory (CADI) refers to a set of tools and guidelines that assist in identifying, collecting, normalizing, correlating, and continuously maintaining all cryptography-related assets, including configurations, dependencies, and usage contexts within a given organization, system, or product scope. It requires both technical tools and management approaches.

Many Post-Quantum transition roadmaps have highlighted the importance of identifying legacy cryptographic assets inside of an organization as a crucial preparatory part. For example:

* The G7 statement on "Advancing a Coordinated Roadmap for the Transition to Post-Quantum Cryptography in the Financial Sector" {{G7}} mentions there should exist six key PQ-migration activities:
    * Awareness and Preparation, **Discovery and Inventory**, Risk Assessment and Planning, Migration Execution, Migration Testing, Validation and Monitoring. The exact six stages are also published in NCCoE analysis on "Six Key Phases of the PQC Migration Journey" {{NCCOEFAQ}}.
* The United States Memorandum for the Heads of Executive Departments and Agencies on "Migrating to Post-Quantum Cryptography" {{OMBM2302}} requires:
    * All federal agencies to "**Submit cryptographic system inventory By May 4, 2023 and annually thereafter**". This important date is also mentioned in CISA directive "Strategy for Migrating to Automated Post-Quantum Cryptography Discovery and Inventory Tools"{{CISAACDI}}.

Although the exact PQ-migration roadmap in each different nations varies, the fact that cryptographic asset discovery and inventory is a key prerequisite for Chief Information Security Officers to asses a migration budget and draft a migration plan does not change. In this draft, we analyze existing CADI methods and potential gaps.

# Terminology {#term}

* CADI (Cryptographic Asset Discovery and Inventory) -- A set of tools and guidelines designed to automate the identification, collection, normalization, correlation, and lifecycle management of an organization's cryptographic assets.
* Cryptographic Asset -- A digital or physical element within a device or service-such as an algorithm, module, configuration, credential, secret, or communication protocol-that protects the confidentiality and integrity of secure data transmission or storage.
* Cryptographic System -- An active software or hardware implementation of one or more cryptographic algorithms that provide one or more of the following services: (1) creation and exchange of encryption keys; (2) encrypted connections; or (3) creation and validation of digital signatures. (according to {{OMBM2302}})
* Cryptographically Relevant Quantum Computers (CRQC) -- A fault-tolerant quantum system capable of solving the underlying hardness assumption problems of widely used public-key cryptography in polynomial time, such as RSA and ECC.

# Problem Statement and Challenges {#ps}

Apart from the milestones and requirements discussed above, there are also a few challenges to identify cryptographic assets:

* Legacy ICT systems were not engineered to treat cryptographic primitives as distinct, trackable assets. Instead, cryptographic implementations are deeply integrated, highly distributed, and hidden across heterogeneous layers of the technology stack.
* Taking inventory manually would take enormous amount of effort, yet there may still be statistical omissions that leave security exposures.
* Methods like Cryptographic Bill of Materials (CBOM) only take effect on new devices to self-announce its cryprographic assets, but it cannot help identify legacy devices and systems.
* Legacy devices and systems often behave like blackboxes, many does not allow installing additional programs or tools on them.

As a result, the ideal CADI tool should meet the following requirements:
*  **Automated** scanning and inventory taking
*  **Agnostic** to target object's system designs and architectures
*  **Integrate** the result to existing network management and operations systems for unified presentation

The requirements for automation, architectural agnosticism, and network management integration suggest a protocol-driven approach, potentially placing this within the scope of the IETF. However, this remains open for discussion.

# Discovery and Identification

There are many ways of doing discovering and identification. We catagorize them into two: active identification and passive identification.

   * **Active Identification**: The case where devices and services modifies its development and/or management process, actively self-announcing their cryptographic assets. Methods falls into this category includes:
       * CBOM Declaration
       * Static Code Scanning
       * Binary / Image Scanning
       * CI/CD Integration
   * **Passive Identification**: The case where devices and services will stay as-is and cannot self-announce their cryptographic assets. These devices will require additional external tools to assist the identification. Methods falls into this category includes:
       * Simulated Handshakes
       * Configuration Extraction
       * Traffic Pattern Analysis
       * Process Identification

## Active Identification

### Cryptographic Bill of Materials (CBOM) and Auxiliary Methods

CBOM-based identification is to empower cryptographic asset owners or vendors to explicitly declare cryptographic usage within their products, components, or services. Usually CBOM declaration is achieved through static code scanning and/or CI/CD integration, so we keep them in the same section.

Scanning Methods:
* **Static Code Scanning**:
    * SCA (Software Composition Analysis): Scan manifest files (like `package.json`, `requirements.txt`, or `pom.xml`) from the codebase to create CBOMS, usually for analyzing third-party libraries dependency.
    * SAST (Static Application Security Testing): Builds a Abstract Syntax Tree (AST) with parsed syntax and data flow of the target source code. It then uses AST to track call chain to end crypto library, track variables (data flow) from source to destination, do reference matching, etc.
        * SCA and SAST is used by IBM CBOMKit-Hyperion.
* **Binary / Image Scanning**:
    * Scan for algorithmic constants, signatures; Extract from OS trust stores and runtime configurations... The process details are omitted in this document due to complexity.
        * Binary / Image Scanning is used by IBM CBOMKit-Theia and CycloneDX Cdxgen.

Tools and Modelling:
* **IBM CBOMkit**, Static code level scanning (Hyperion) and Artifact level scanning (Theia) {{IBMCBOM}}: uses `implements` and `uses` relationship to create dependency diagram.
    * `implements`: Describes the list of protocols or algorithms this module implements.
    * `uses`: Describes what modules are used by this service.
    * For example, Application Nginx `uses` Library `libssl.so` that `uses` Protocol TLS v1.3/v1.2 that `uses` `libcrypto.so` that `implements` Algorithm MD5, SHA256, AES-128-GCM.
* **CycloneDX Cdxgen**, Artifact level scanning {{CDXGEN}}: uses `dependsOn` and `provides` relationship to create dependency diagram.
    * `dependsOn`: Describes other modules or services this module depends on.
    * `provides`: Describes all capabilities this module provides.
    * For example, Application Nginx `dependsOn` Library `libssl.so` , which `provides` TLS 1.2.

After scanning, the result will be recorded as a CBOM object.

CBOM data specification: A standardized cryptographic asset object includes cryptographic algorithms, digital certificates, protocols, private keys, public keys, cryptographic keys, ciphertext information, digital signatures, digests (the output values of hash functions), initialization vectors (input parameters for encryption algorithms), seeds, salts, shared secrets, authentication tags, passwords, credentials, and tokens. For a detailed specification, see CycloneDX SBOM v1.6 https://cyclonedx.org/news/cyclonedx-v1.6-released/

~~~
"components": [
 {
  "name": "google.com",
  "type": "cryptographic-asset",
  "bom-ref": "crypto/certificate/google.com@sha256:1e15e0fbd3ce9...",
  "cryptoProperties": {
    "assetType": "certificate",
    "certificateProperties": {
      "subjectName": "CN = www.google.com",
      "issuerName": "C = US, O = Google Trust ... LLC, CN = GTS CA 1C3",
      "notValidBefore": "2016-11-21T08:00:00Z",
      "notValidAfter": "2017-11-22T07:59:59Z",
      "signatureAlgorithmRef": "crypto/algorithm/sha-512-rsa@1.2.840..",
      "subjectPublicKeyRef": "crypto/key/rsa-2048@1.2.840.113549.1.1.1",
      "certificateFormat": "X.509",
      "certificateExtension": "crt"
    }
  }
~~~
*Figure 1: Example CBOM*

Gaps: Obviously, this does not help with identifying the mass legacy devices, components or services.

## Passive Identification

### Simulated Handshakes

Simulated handshake is a network-probing technique where a discovery tool acts as a client and intentionally initiates cryptographic protocol negotiations (e.g., TLS, SSH, IPsec) with enterprise network endpoints without completing the full data session. During the negotiation process, the discovery tool determines the specific cryptographic protocol versions, negotiation mechanisms, and ciphersuites supported by the endpoint. This is also known as *Active Probing*.

Steps of simulated handshake include:
1. **Port Scan:** The tool scans for typical ports for specific secure communication protocols (443 for HTTPS, 22 for SSH, etc)
2. **Probing:** The tool generates a RFC-compliant connection initialization packet, including all currently standardized ciphersuites, and examine the response.
3. **Response Extraction:** The target object respond with supported ciphersuites, following its own internal priority rules.
4. **Parsing:** The tool intercepts this raw response and decodes it as cryptographic assets.

This works for most secure protocols that includes negotiation:
* IPSEC
* TLS
* SSH

Pros and Cons/Gaps:
* Strengths:
    * This kind of method does not require complex engineering refactoring. Since it is agnostic to the design of the target, it works best for the case where the manager operates vast heteogenous devices from different vendors, versions and locations, hence telecommunications case.
    * This probing is one-time, thus it will not create unhandleable amount of probing packets.
* The limitation of this method is that
    * Some masking mechanism will stop the probing (firewalls, load balancers, port-concealing protocols like Single Packet Authorization that enforce a default drop for unexpected packets), leaving a hidden security posture inside of the system.
    * Network management cannot know local application components that rests internally in the device and are not exposed over a socket.

Tools:
* Cisco Mercury project {{MERCURY}} provides an open source packet capture and analysis tool. It can read network packets, identify metadata of interest, and write out the metadata (including cryptographic-related) in JSON format.

### Traffic Pattern Analysis

Unlike simulated handshakes that actively query an endpoint, Traffic Pattern Analysis listens passively to live network traffic at strategic aggregation points. It infers cryptographic asset usage from protocol metadata, statistical characteristics, and behaviors without decrypting the underlying data payload.

Methods of Traffic Pattern Analysis includes:
* Listen to unencrypted negotiations like `clientHello`/`serverHello`, Server Name Indication (SNI), response of an unencrypted server digital certificate request, etc.
* Listen to handshake fingerprints and compare them to existing patterns, as protocol implementations and behaviors are often rigid.
* Listen to packet length and arrival time, do statistical pattern recognition.

Pros and Cons/Gaps:
* Strengths:
    * Similar to Simulated Handshakes, this kind of method is not intrusive and is agnostic to the design of the target.
* The limitation of this method is that
    * Negotiation information like `clientHello` {{RFC9849}} and SNI are tending to be encrypted.
    * Protocols tend to have constant size packets {{RFC9347}}.
    * Alternative routing: network requests/responses could exit through a different routing path.

### Process Identification

Process Identification requires Endpoint Detection and Response (EDR) installed to have cryptographic visibility inside of a system.

Methods of Process Identification includes:
* Kernel-Level System Call Auditing by deploying eBPF programs into the kernal space. These filters can capture kernel events to determine exactly which process (down to PID) have initiated an encrypted network communication.
* Shared Library and Binary Hooking, by monitoring process load table to see if cryptographic dynamic libraries are loaded into the memory space.
* Live Memory Scanning, by periodically scanning volatile memory for indication of creation of keys or signatures (as high-entropy memory segments), and then compare them with known patterns.

Pros and Cons/Gaps:
* Strengths:
    * Have deeper visibility for cryptographic assets within devices.
* The limitation of this method is that
    * Requires loading new software (at least EDR/RDR) to legacy devices.
    * Potential blind spots due to lower pattern coverage.

### Configuration Extraction

The Configuration Extraction is a bit similar to the source code scanning. This section focuses more on the runtime configurations. In different use cases, the configuration extraction methods differs significantly. This section is to be extended.

Methods of Configuration Extraction includes:
* In network device management, NETCONF and YANG provide a standardized, model-driven management interface that enables programmable configuration and state validation. The network managing platform can define a YANG model and extract configurations from a device.
* In cloud-native development, general `.yaml` configuration files, `.env` environment variables files, Kubernetes secret manifest, etc, can be extracted by administrative scripts or through management interfaces (e.g., AWS CloudControl, Kubernetes API).

# Inventory

TBD: After discovery, the result should best be presented in a unified network management platform for a comprehensive view. The Network Inventory (IVY) working group is working on this topic and cryptographic properties could become extensions to IVY records. The details of this section could invite more network management experts for input.

# Potentially Related Working Groups

* IVY -- extend network management views with cryptographic attributes.
* SCITT -- extend SBOMs to adapt CBOMs.
* PQUIP -- as a migration guideline.

# Tools Compilation

Due to limited time, compiling existing CADI tools remain open and will be done in next version.

# Security Considerations

This document has no further security considerations.

# IANA Considerations

This document has no IANA actions.

--- back
