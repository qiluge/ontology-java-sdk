# ONTID 2.0 Java SDK

## Role

owner: claim owner, who hold some claims and create presentation;

issuer: claim issuer, who issue claim;

verifier: claim verifier, who verify claim and presentation.

## Class Specification

Refer: https://www.w3.org/TR/vc-data-model/#basic-concepts

### CredentialStatus

[code](./VerifiableCredential.java#L30)

Refer: https://www.w3.org/TR/vc-data-model/#status

CredentialStatus.id: claim contract address;

CredentialStatus.type: constant value, "Claim Contract".

### VerifiableCredential

[code](./VerifiableCredential.java)

Refer: https://www.w3.org/TR/vc-data-model/#basic-concepts

VerifiableCredential.issuer: issuer ontId.

### Proof

[code](./Proof.java)

Refer: https://www.w3.org/TR/vc-data-model/#proofs-signatures

Proof.type: only use "EcdsaSecp256r1Signature2019" at currently;

Proof.proofPurpose: only use "assertionMethod" at currently;

Proof.verificationMethod: pubkey uri, like:"did:ont:AVe4zVZzteo6HoLpdBwpKNtDXLjJBzB9fv#keys-2";

Proof.signature: hex-encoded ontology signature.

### VerifiablePresentation

[code](./VerifiablePresentation.java)

Refer: https://www.w3.org/TR/vc-data-model/#presentations-0

VerifiablePresentation.holder: holder ontId.

### SignRequest

[code](./SignRequest.java)

Claim owner send `SignRequest` to issuer to create claim.

### OntIdPubKey

[code](./OntIdPubKey.java)

It's a utility class, represent one public key of ontId.

### OntIdSigner

[code](./OntIdSigner.java)

It's a utility class, represent ontId signer.


### OntId2

[code](./OntId2.java)

It's ONTID2 SDK class, all ontId 2.0 function should entry from here.

OntId2.ClaimRecord: claim contract;

OntId2.OntId: ontId contract.

## Interface

### Constructor

1. **public OntId2(String ontId, Account signer, ClaimRecord claimRecord, OntId ontIdContract)**

    Generate OntId2 object.

    If `ontId` not empty and `signer` not null, there will query `OntIdPubKey` from ontology chain and use it as self signer.
    
    So owner and issuer should set `ontId` and `signer` while create OntId2 object, and verifier may not set those.

2. **public void updateOntIdAndSigner(String ontId, Account signer)**

    Update `ontId` and `signer` account.

### GenSignRequest and VerifySignRequest

1. **public SignRequest genSignReq(String claim, boolean hasSignature)**

    * calculate claim hash256 bytes;
    * if hasSignature, use self `signer` to sign hash;
    * return SignRequest object.

2. **public boolean verifySignReq(SignRequest req)**

    * use `req.ontId` to verify `req.signature` of `req.claim`.

### Create Claim

Issuer create claim and commit claim hash to ontology chain.

1. **public VerifiableCredential createClaim(String[] context, String[] type, Object credentialSubject, Date expiration)**

    * create VerifiableCredential object;
    * check `expiration`;
    * generate `Proof`;
    * calculate `VerifiableCredential.id`.

2. **public String commitClaim(VerifiableCredential claim, String ownerOntId, Account payer, long gasLimit, long gasPrice, OntSdk sdk)**

    * create commit claim tx;
    * use self.signer and `payer` to sign tx;
    * send tx to ontology chain.

### Verify Claim

Verifier could verify claim.

The process consists of 4 aspects.

1. **public boolean verifyClaimOntIdCredible(String[] credibleOntIds, VerifiableCredential claim)**

    * check `claim.proof.verificationMethod` is `claim.issuer`;
    * check `claim.proof.verificationMethod` is existed in `credibleOntIds`.

2. **public boolean verifyClaimNotExpired(VerifiableCredential claim)**

    * check claim not expired.

3. **public boolean verifyClaimSignature(VerifiableCredential claim)**

    * check `claim.proof.verificationMethod` is `claim.issuer`;
    * use `claim.proof.verificationMethod` as public key to verify `claim.proof`.

4. **public boolean verifyClaimNotRevoked(VerifiableCredential claim)**

    * query status of claim;
    * check status equals `01`.

### Create Presentation

Refer: https://www.w3.org/TR/vc-data-model/#presentations-0.

Owner could create presentation by using one or multi `VerifiableCredential`.

**public VerifiablePresentation createPresentation(VerifiableCredential[] claims, String[] context, String[] type, OntIdSigner[] otherSigners, String holderOntId)**

* create `VerifiablePresentation` object;
* use self signer to generate first proof;
* use `otherSigners` to generate other proofs;
* calculate id.

### Verify Presentation

Verifier could verify presentation.

**public boolean verifyPresentationProof(VerifiablePresentation presentation, int proofIndex)**

* check proof of `presentation`.


## Claim Record Contract

[code](../smartcontract/neovm/ClaimRecord.java)

OntId 2.0 use a new version ClaimRecord contract. There are 4 new interface.

### Commit

**public String sendCommit2(String issuerOntid, String password, byte[] salt, String subjectOntid, String claimId, int pubkeyIndex, Account payerAcct, long gaslimit, long gasprice)**

Used to commit a claim id to ontology chain.

### Revoke

**public String sendRevoke2(String ownerId, String password, byte[] salt, String claimId, int pubkeyIndex, Account payerAcct, long gaslimit, long gasprice)**

Used to revoke a recorded claim id.

### Remove

**public String sendRemove2(String ownerId, String password, byte[] salt, String claimId, int pubkeyIndex, Account payerAcct, long gaslimit, long gasprice)**

Used to remove a recorded claim id.

### GetStatus

**public String sendGetStatus2(String claimId)**

Used to query claim status by claim id, there are 3 return value:

* "00": revoked.
* "01": committed;
* "02": removed;

## Compatibility

Considering that there are a lot of old version of claim, the protocol upgrade must be compatible with old data. The main
difference between the new and old protocol is in the form of claim.

The old ONTID protocol use [OntId](../smartcontract/nativevm/OntId.java) to generate and verify claim. However, the new
use [OntId2](./OntId2.java) to generate and verify claim. 

In the other hands, we also provide the methods that verify old claim by new ClaimRecord contract:

* **public boolean verifyClaimOntIdCredible(String claim, String[] credibleIds)**
* **public boolean verifyClaimNotExpired(String claim)**
* **public boolean verifyClaimSignature(String claim)**
* **public boolean verifyClaimNotRevoked(String claim)**

See these code at [here](../smartcontract/nativevm/OntId.java#L2186-L2258).

## Demo

There are some code to illustrate how to use ONTID 2.0 sdk.

[code](../../../../demo/OntId2Demo.java)