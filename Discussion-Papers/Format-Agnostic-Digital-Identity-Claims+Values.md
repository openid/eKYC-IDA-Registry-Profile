#
**Date:** March 2, 2026
**Status:** DRAFT for Working Group Review
**Subject:** Format-Agnostic Digital Identity for Cross-Sector Compliance & Examiner Defense
**Author:** Juliana Cafik
**Audience:** OIDF Digital Credentials Protocols (DCP) WG, eKYC/IDA WG

## 1. Purpose

Verifiable Digital Credentials (VDCs), such as mobile driver’s licenses (mDLs), are an emerging technology that can improve current know your customer (KYC) processes. However, to transition to this new form of identity evidence, relying parties require greater clarity around the assurance of the credential being presented and the functional ability to collect the information necessary to satisfy stringent compliance, risk management and audit requirements. To address these needs, this discussion paper proposes a globally interoperable framework that defines identity claims designed to support high-assurance KYC processes and the use of VDCs as identity evidence. By defining standardized claims the framework empowers institutions to execute enhanced, runtime risk decisions aligned to the risk profile of a specific transaction or account type. To meet stringent examiner scrutiny, the framework ensures standardized claims and parameter values for defensible audit trails, providing institutions with clear machine-readable evidence of what was verified and the cryptographic proof necessary to seamlessly fulfill ongoing recordkeeping mandates.    
  
Threat Mitigation Consideration: The architectural requirements defined herein, specifically outline the cryptographic holder binding and the transmission of raw binary evidence as foundational artifacts for KYC but also provide defences against a rapidly expanding attack surface. As threat actors deploy hyper-efficient, agentic AI-driven deepfakes and scaled synthetic identity injections, relying on semantic strings (e.g., asserting a face was checked) is inadequate. This proposed profile also proposes to enforce mathematical, cryptographic proof to secure the ecosystem against next-generation automated exploitation.

## 2. Scope

This paper proposes a protocol, format, and jurisdiction-agnostic framework for identity claims that extends the: [OpenID Identity Assurance Schema Definition 1.0 ](https://openid.net/specs/openid-ida-verified-claims-1_0-final.html) and [OpenID Connect for identity Assurance Claims Registration 1.0 ](https://openid.net/specs/openid-connect-4-ida-claims-1_0-final.html)to directly support Relying parties (RPs) in establishing a legally defensible “reasonable belief” of customer identity. It introduces a standardized parameter registry with defined URNs and normalizes the claims required for institutions to capture and maintain rigorous, examiner-ready requirements. It addresses claims provenance, distinguishing between issuer-attested and wallet-asserted claims to facilitate dynamic, risk-based decisioning during the onboarding process. Additionally, the framework covers security considerations for claim verification to mitigate advanced presentation threats.

## 3. Protocol and Format Agnosticism
Previous iterations of identity schema mapping created friction by forcing format collisions (e.g., attempting to translate binary CBOR/COSE mDL data into plaintext JSON/JOSE arrays), which inherently breaks the trusted credential issuer's digital signature. 

To achieve true format agnosticism, implementations MUST separate theTrust Establishment Layer (the registry/VICAL), the Metadata Envelope (the compliance vouching protocol), and the Cryptographic Payload (the native evidence). 
* **Transport Layer Agnostic:** Supports ISO 18013-5/7, OpenID4VP, pure OIDC4IDA, or emerging decentralized routing protocols.
* **Format Agnostic Evidence:** The schema acts strictly as a standardized compliance and metadata "Cover Sheet". The underlying evidence MUST be transported in its native, unaltered cryptographic format (e.g., ISO 18013-5 Mobile Security Objects, W3C Verifiable Credentials in JSON-LD/BBS+, or SD-JWTs).

## 4. Normative Relying Party (RP) Processing Sequence
To ensure jurisdictional independence, prevent the acceptance of transcoded, unverified text, and strictly comply with the ISO 23220-4 Presentation Phase architecture, the Relying Party (acting as the mdoc Reader) MUST execute validation in the following sequential order.

The verification of the metadata envelope MUST NOT proceed until the native cryptographic proofs of the underlying payload are validated.

4.1 Trust Resolution (Verifier Identity Certificate Authority List)
Before evaluating a presentation, the mdoc Reader MUST establish local trust independent of the transport connection.
Action: The Reader resolves the public key material from a recognized Trust Anchor or Entity Configuration (e.g., an ISO-compliant VICAL). This caches the Issuing Authority Certificates (IACA) required for offline or zero-trust validation.

4.2 Device Engagement and Reader Authentication
During the initial presentation request, the mdoc Reader MUST assert its own identity and intent to the credential holder to mitigate cross-sector credential replay and establish a secure, authenticated channel.
Action: The Reader presents its cryptographic Reader Certificate (provisioned via the Trust Registry). The holder’s wallet validates this certificate against its local policies before releasing the identity evidence.

4.3 mdoc Authentication (Cryptographic Verification)
Upon receiving the presentation payload (e.g., via Device Retrieval or Network Retrieval), the mdoc Reader MUST execute the dual-verification process defined by ISO 18013-5 and ISO 23220-4 to validate the Mobile Security Object (MSO).
Issuer Authentication: The Reader isolates the embedded binary attachments and verifies the MSO’s digital signature against the cached IACA public keys (resolved in step 3.1). This mathematically proves the provenance of the data and ensures it has not been altered since issuance.

Device Authentication: The Reader verifies the cryptographic proof (e.g., a signature over the session transcript generated by the device's secure element) to ensure the mdoc is bound to the specific hardware presenting it. This provides the foundational defense against synthetic media and deepfake injections.

4.4 Data Extraction and Policy Correlation
Only after mdoc Authentication is mathematically proven successful may the mdoc Reader process the presentation for business logic.
Action: The Reader extracts the raw verifiable data (e.g., CBOR tags) and correlates it with the standardized plaintext metadata mapped in the overarching assurance schema (e.g., evaluating the assurance_classification and context_uri). The correlated, finalized payload is then committed to the verifier's internal audit log to satisfy the "Examiner Defense."

## 5. The Six Pillars: Jurisdictionally Independent Compliance/Risk Decision Elements
To enable global scalability, all jurisdictional parameters are abstracted using a standardized registry of Uniform Resource Names (URNs) and nested context definitions.

**Pillar I: Provenance & Jurisdictional Assurance**
* **Requirement:** Verify the credential was issued under recognized, government-mandated proofing standards (e.g., US NIST IAL2, EU eIDAS High, AU TDIF IP3, NZ DISTF High).
* **Implementation:** Utilize `context_uri` to define the governing legal framework and `assurance_classification` to assert the credential's compliance status within that specific framework.

**Pillar II: Attribute Completeness (National Identifiers)**
* **Requirement:** Verify jurisdictional tax, civic, or national identification numbers without requiring plaintext transmission or storage.
* **Implementation:** Implement a `national_identifier_match` object detailing the country context and identifier type (e.g., US TIN, EU PID, AU TFN, NZ IRD) to provide a generic, cryptographic boolean match signal.

**Pillar III: Cryptographic Holder Binding (Binary Evidence)**
* **Requirement:** Provide mathematical proof that the presenter is the legitimate owner of the credential and the authorized device.
* **Implementation:** Implementations MUST embed **two** distinct native attachments:
    1.  **Identity Provenance:** The raw, unaltered credential (e.g., `application/cbor` for mDLs or `application/vc+jwt` for W3C VCs). *Omitting this nullifies the Examiner Defense*.
    2.  **Device Binding:** The Hardware Key Attestation or dynamic Session Transcript (e.g. mode Reader Authentication or verifiable presentation signatures).

**Pillar IV: Freshness & Revocation**
* **Requirement:** Ensure the identity has not been revoked, suspended, or altered ("Reasonable Belief”).
* **Implementation:** Assert the `check_method` (e.g., a cryptographic Status List check or `"revocation_freshness_check”` from a wallet) and utilize standardized URNs for the `organization` serving as the authoritative registry for the authoritative status check.

**Pillar V: Data Lineage & Auditability (The Examiner Defense)**
* **Requirement:** Reconstruct the complete identity event for regulatory recordkeeping.
* **Implementation:** RPs satisfy strict auditability by persistently storing the complete, finalized payload (the metadata cover sheet and its embedded native binary attachments) entirely internally. This record MUST be indexed securely by the transaction `session_id`.

**Pillar VI: Transaction Context & Consent**
* **Requirement:** Bind the identity verification event to the specific operational intent (e.g., FinCEN CDD, GDPR purpose limitation, or CDR consent).
* **Implementation:** Inject `account_intent` and a verifiable `consent_id` directly into the verification object.

## 5.1 Security Consideration: Audience Binding (aud)
Because this profile acts independently of an OIDC network layer, standard token audience bindings (aud) are insufficient. The presentation envelope MUST establish cryptographic Verifier Authentication. Whether utilizing mTLS, ISO 18013-5 Reader Certificates extracted from the VICAL, or a Verifiable Presentation signature signed over a verifier-provided nonce, the system MUST ensure the payload was explicitly generated for the specific verifier, mitigating cross-sector replay attacks.

## 5.2 Security Consideration: Wallet Integrity and Attestation 
While Pillar III establishes the cryptographic binding of the credential to a specific device keypair, the Relying Party (RP) MUST also establish trust in the software and hardware environment executing the presentation. To defend against credential extraction, software emulators, and rogue application spoofing, the presentation envelope MUST include cryptographic proof of the wallet's integrity.

Implementations SHOULD mandate the inclusion of OS-level or hardware-backed Wallet Attestation (e.g., Apple App Attest, Android Play Integrity, or hardware keystore attestations) within the metadata envelope. This attestation mathematically proves that the wallet application is authentic, unmodified, and operating within a secure execution environment (such as a Trusted Execution Environment or Secure Enclave). The RP MUST validate this attestation against recognized publisher registries or policy thresholds before processing the underlying cryptographic payload, ensuring the zero-trust architecture extends to the edge device.

## 6. Standardized Parameter Value Registry (Normative)
To enable true jurisdictional agility, implementations MUST map local compliance terminology to standardized URNs and enumerated values. This ensures a Verifier’s policy engine can programmatically evaluate risk without needing to parse bespoke strings from every global issuing authority.

### 6.1 Assurance Context URIs (`context_uri`) EXAMPLES
Defines the specific legal or regulatory trust framework governing the verification event.

| Jurisdiction | Parameter Value (`context_uri`) | Governing Framework | Description |
| :--- | :--- | :--- | :--- |
| **US** | `urn:openid:assurance:us:real_id` | DHS REAL ID Act | Federal standards for state-issued driver's licenses. |
| **EU** | `urn:openid:assurance:eu:eidas` | eIDAS 2.0 Regulation | European digital identity framework (EUDI Wallet). |
| **AU** | `urn:openid:assurance:au:tdif` | TDIF | Australian Trusted Digital Identity Framework. |
| **NZ** | `urn:openid:assurance:nz:distf` | DISTF | New Zealand Digital Identity Services Trust Framework. |
| **Global** | `urn:openid:assurance:global:iso` | ISO/IEC 29115 | Global entity authentication assurance framework. |

### 6.2 Assurance Classifications (`assurance_classification`)
Values are dependent on the associated `context_uri` and define the specific level of assurance achieved.

| Associated Context | Parameter Value (`issuance_assurance_classification`) | Definition |
| :--- | :--- | :--- |
| **US** | `status:full_compliance:us:real_id` | Credential meets highest mandated federal standards. |
| **EU** | `loa:high:eu:eidas` | Cryptographic hardware binding verified; equivalent to face-to-face. |
| **AU** | `loa:substantial:au:tdif` | Strong authentication, but lower initial proofing requirements. |
| **AU** | `ip:3:au:tdif` | Identity Proofing Level 3 (High Confidence) under AU TDIF. |
| **NZ**`|`level:high:nz:distf` | High assurance identity proofing under NZ DISTF. |

### 6.3 Wallet Assurance Classification (`wallet_assurance_classification`) 
Values are dependent on the associated `context_uri` and define the specific security, cryptographic binding, and certification level of the digital wallet storing and presenting the credential.

| Associated Context | Parameter Value (`issuance_assurance_classification`) | Definition |
| :--- | :--- | :--- |
| **US** | `storage:hardware_backed` | Credential keys are bound to a device-native secure hardware enclave (e.g. Secure Element), aligning with ISO mDL requirements.|
| **EU** | `wallet:wsce_certified` | Wallet operates in a certified Wallet Secure Cryptographic Environment (WSCE) meeting high EUDI Wallet security requirements. |
| **AU** | `cl:3` | Credential Level 3 (High) under AU TDIF, requring cryptographic hardware binding and device-level security. |
| **NZ** | `storage:secure_zone` | Keys are protected by a mobile device hardware-backed keystore in compliance with DISTF technical standards. |
| **Global** | `aal:3` | Authenticator Assurance Level 3 (NIST), requiring hardware-based cryptographic proof of possesion and strong resistance to verifier impersonation. |

### 6.4 Audience Assurance Classification (`audience_assurance_classification`) 
Values are dependent on the associated `context_uri` and define the trust, vetting, or acceditation status of the (Relying Party) requesting the data.

| Associated Context | Parameter Value (`issuance_assurance_classification`) | Definition |
| :--- | :--- | :--- |
| **US** | `verifier:federally_recognized` |Relying Party is a vetted federal or state agency with authorization to request high-assurance identity data.|
| **EU** | `rp:certified` | Verifier is registered, cryptographically authenticated, and authorized to request specific data under the eIDAS trust list |
| **AU** | `rp:accredited` | Relying Party has undergone formal accreditation and compliance auditing under the Australian TDIF. |
| **NZ** | `rp:trusted_participant` | Verifier is an officially approved and onboarded participant within the New Zealand DISTF ecosystem. |
| **Global** | `verifier: mutually_authenticated` | verifier identity abd authorization have been established via a globally recognized trust registry or X.509 PKI infrastructure. |

### 6.5 National Identifier Types (`identifier_type`)
Standardized values for the `national_identifier_match` object.

| Jurisdiction | Parameter Value (`identifier_type`) | Jurisdictional Target |
| :--- | :--- | :--- |
| **US** | `ssn` | Social Security Number |
| **US** | `tin` | Taxpayer Identification Number |
| **EU** | `pid` | Personal Identification Data (eIDAS) |
| **AU** | `tfn` | Tax File Number |
| **AU** | `crn` | Customer Reference Number (Centrelink) |
| **NZ** | `ird` | Inland Revenue Department Number |

### 6.6 Authoritative Organizations (`organization`)
Specifies the entity that performed the freshness or revocation check.

| Jurisdiction | Parameter Value (`organization`) | Entity Type / Example |
| :--- | :--- | :--- |
| **US** | `urn:issuing_authority:us:aamva` | American Association of Motor Vehicle Administrators (DTS) |
| **EU** | `urn:issuing_authority:eu:es:fnmt` | Fábrica Nacional de Moneda y Timbre (Spain - National Mint/Issuer) |
| **EU** | `urn:issuing_authority:eu:member_state` | Generic routing for EU Member State national registries |
| **AU** | `urn:issuing_authority:au:dvs` | Australian Document Verification Service |
| **NZ** | `urn:issuing_authority:nz:dia` | New Zealand Department of Internal Affairs |
| **NZ** | `urn:issuing_authority:nz:nzta` | New Zealand Transport Agency (Waka Kotahi) |

---

## 7. Appendix A: Format-Agnostic Conformance Example
This payload demonstrates the assurance schema acting as a pure metadata envelope for native, jurisdictionally independent cryptographic evidence. Noting mandatory OIDC routing claims (iss, sub, aud) are replaced by explicit trust binding. This example illustrates an Australian TDIF verification context.

```json
{
  "_comment": "PROTOCOL-AGNOSTIC PRESENTATION BINDING",
  "verifier_id": "did:web:relying-party.example.com", 
  "presentation_time": "2026-03-03T12:00:00Z",
  "session_nonce": "a1b2c3d4e5f6g7h8",

  "_comment": "FORMAT-AGNOSTIC EKYC METADATA ENVELOPE",
  "verified_assurance": {
    "verification": {
      "trust_framework": "urn:assurance:global:iso",
      "assurance_level": "loa-3",
      
      "_comment": "TRANSACTION CONTEXT EXTENSIONS",
      "account_intent": "high_value_wire_transfer",
      "consent_id": "consent-ref-98765-xyz",
      
      "assurance_details": [
        {
          "_comment": "JURISDICTION-AGNOSTIC ASSURANCE ROUTING",
          "assurance_type": "identity_proofing",
          "context_uri": "urn:assurance:au:tdif", 
          "assurance_classification": "ip:3" 
        }
      ],
      "evidence": [
        {
          "type": "document",
          "evidence_format": "application/cbor",
          "revocation_freshness_check": "2026-03-03T19:24:18Z",
          "payload": "<Base64 encoded raw MSO/mdoc data block>"
        }
      ]
    }
  }
}
```

---

## 8. Appendix B: Cryptographic Payload Mapping (mdoc CBOR to JSON Schema)
To maintain the "Examiner Defense" and zero-trust verification principles, the protocol-agnostic schema MUST NOT break the native digital signatures of the source credential. When handling ISO/IEC 18013-5 or 18013-7 mobile documents (mdocs), Relying Parties must treat the CBOR (Concise Binary Object Representation) data as an immutable artifact.

The schema does not replace the CBOR data; rather, it acts as a routing wrapper that extracts high-level metadata (such as validity windows and document types) to populate the JSON evidence array for standard policy evaluation, while carrying the raw binary for deep cryptographic verification.

Conformance Implementation Example: When a Verifier's policy engine processes an mdoc, the resulting extracted evidence object MUST reflect the following structure. The payload claim securely transports the raw evidence to satisfy compliance without forcing the JSON processor to decode the entire CBOR tree immediately.

JSON

{
  "evidence": [
    {
      "type": "document",
      "evidence_format": "org.iso.18013.5.1.mDL",
      "not_valid_before": "2023-01-15T08:00:00Z",
      "not_valid_after": "2028-01-14T23:59:59Z",
      "revocation_freshness_check": "2026-03-01T12:00:00Z",
      "device_binding_verified": true,
      "_comment": "Base64 encoded DeviceResponse containing issuerAuth and deviceAuth",
      "payload": "o2h2ZXJzaW9uZTEuMGZkb2N1bWVudHOCo2dkb2NUeXBl..." 
    }
  ]
}
