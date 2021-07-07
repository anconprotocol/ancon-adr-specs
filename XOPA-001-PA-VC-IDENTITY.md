
# 
# Panamanian Identity Verified Credential Spec and DID


<table>
  <tr>
   <td><strong>Author</strong>
   </td>
   <td>Rogelio Morrell, Ruben Guevara
   </td>
  </tr>
  <tr>
   <td><strong>Category</strong>
   </td>
   <td>VC, DID
   </td>
  </tr>
  <tr>
   <td><strong>Created</strong>
   </td>
   <td>2021-07-07
   </td>
  </tr>
</table>



##   Simple Summary

Implement a DID/VC for Panama citizen centralized identity as proof of identity and combined with proof of humanity protocols (BrightID, Proof of Humanity)

##  Abstract

We propose a QR readable web component that is able to read national id cards, generate an unique seed randomization use to create a keypair.  With a VC model 
based on the id card, we create both DID and VC DIDDoc.


##  Motivation

##  Prior Art

## Specification


###  handleQR

```typescript
async handleQRFromImage() {
    let response = {};
    try {
      const codeReader = new BrowserQRCodeReader();
      const img = await PromiseFileReader.readAsDataURL(this.files);
      const imgSrc = document.createElement("img");
      imgSrc.src = img;
      document.body.appendChild(imgSrc); // we need to append the element to the dom -> otherwise it will not work in firefox
      const result: any = await codeReader.decodeFromImage(imgSrc);
      imgSrc.remove();
      response = {
        name: result.text.split("|")[1].split(" ")[0],
        lastname: result.text.split("|")[2].split(" ")[0],
        data: result.text
      };
    } catch (e) {
      return {};
    } finally {
    }
    return response;
  }
```

### issueCredential

```typescript
const cedula = new CedulaVC();
    const creds = await cedula.issueCredential(
      { name, lastname, hash },
      {
        issuer: did.issuer,
        resolver: did.resolver,
        address: did.preferredDIDAddress,
        ethereumAddress: ethereum.account
      }
    );
  ```
  
  
  ### createVerifier
  
  ```typescript
  async shareVerifier() {
    const { ethereum, did, storage } = this.solidoProps;
    const cedula = new CedulaVC();
    let jwt = await storage.getIdCredential();
    const verifier = await cedula.createVerifier(jwt, {
      address: did.preferredDIDAddress,
      ethereumAddress: ethereum.account,
      resolver: did.resolver,
      issuer: did.issuer
    });
    const ref = await storage.setBinaryData(verifier, "text/plain");
    const data = {
      title: "Sharing my ID card verifier",
      text: "ID Card Verifier",
      url: `${location.host}/#/verify_credentials/${ref}`
    };
    await (navigator as any).share(data);
  }

  ```
  
  
### verify (verify presentation)  
  
 ```typescript
 
  async verify() {
    const { resolver } = this.solidoProps.did;
    const cedula = new CedulaVC();
    const ref = this.verificationPresentation;
    const verifier = await cedula.verify(this.verificationPresentation, {
      resolver
    } as any);
    this.hasVerify1 =
      verifier.signer.owner === this.creds.payload.iss ? "green" : "red";
  }
  ```
  
 ### VC Model
 
 ```
  
{
    "@context": {
      "@version": 1.1,
  
      "idpa": "https://vc.auth2factor.com/panama-tribunal-electoral-id-credential/v1#",
      "xsd": "http://www.w3.org/2001/XMLSchema#",
  
      "name": "idpa:name",
      "familyName": "idpa:familyName",
      "hash": "idpa:hash"
  
    }
  }
  ```
  
  
  ### Reference Implementation
  
  ```typescript
  export class CedulaVC {
  async issueCredential(id: NationalIDCard, did: DidConfig) {
    const vc = {
      "@context": [
        "https://www.w3.org/2018/credentials/v1",
        "https://vc.auth2factor.com/panama-tribunal-electoral-id-credential/v1"
      ],
      type: ["VerifiableCredential", "PanamaTribunalElectoralIDCredential"],
      credentialSubject: {
        //  id: `https://vc.auth2factor.com/panama-tribunal-electoral-id-credential/${did.address}:cedula`,
        name: id.name,
        familyName: id.lastname,

        hash: keccak256(utils.toUtf8Bytes(id.hash))
      }
    };

    const vcPayload: VerifiableCredentialPayload = {
      sub: did.address,
      nbf: moment().unix(),
      vc
    };
    const vcJwt = await createVerifiableCredential(
      vcPayload,
      did.issuer,
      did.address
    );

    const verifiedVC = await verifyCredential(vcJwt, did.resolver);

    return vcJwt;
  }

  verify(jwt: string, did: DidConfig) {
    return verifyPresentation(jwt, did.resolver);
  }

  async createVerifier(jwt: string, did: DidConfig) {
    const vpPayload: PresentationPayload = {
      exp: moment()
        .add(15, "minutes")
        .unix(),
      audience: location.host,
      vp: {
        "@context": ["https://www.w3.org/2018/credentials/v1"],
        type: ["VerifiablePresentation"],
        verifiableCredential: [jwt]
      }
    };
    const vpJwt = await createPresentation(vpPayload, did.issuer, did.address);
    return vpJwt;
  }
}

  ```
  
  
  
  
  
  
  



##  Test Cases


##  Copyright

Copyright Industrias de Firmas Electrónicas, S.A. (IFESA), 2021.
