Q: Meta-question; are the uses of 'must', 'should' etc here to be interpreted per [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) ?

Q: Meta-question; these requirements encompass a fairly large scope. I believe that we can spec and ship SPC in 'chunks' towards this goal, where each stage might add a few more capabilities meeting these requirements, but not have every single requirement in v1. Do others agree?

# Secure Payment Confirmation: Requirements and Design Considerations

Status: This is a draft document without consensus.

Though this document we seek to build consensus around requirements
and design considerations for SPC.

***Note:*** These requirements are not prioritized.

See also: [SPC Scope](scope.md) for definitions and more information.

## Feature Detection

* It must be possible to determine programmatically whether a browser supports SPC.

## Web Context Support

* It must possible to invoke SPC from an iframe. See [issue 68](https://github.com/w3c/secure-payment-confirmation/issues/68).
* It must possible to invoke SPC from within Payment Request API.
* It must possible to invoke SPC from within a Payment Handler.

## Enrollment

* It must be possible to enroll an SPC Credential outside of a transaction.
* It must be possible to enroll an SPC Credential during a transaction. This enrollment should not prevent timely completion of the transaction.
* It must be possible to enroll an SPC Credential from code on a merchant site.
* It must be possible to enroll an SPC Credential from a payment handler.
* Each browser should natively support an SPC credential enrollment user experience.

### Instrument Information

* Enrollment must include display information for the associated instrument.

Q: Should this be a 'must' if we're allowing for per-RP credentials (i.e. as opposed to requiring per-payment-instrument)? I could see it as an optional include in that case.

* Because instrument display information is available to the relying party (e.g., provided by the relying party itself, the merchant, or some other party), it is not a requirement that this information be stored in the browser as part of the SPC Credential.

## Transaction Confirmation User Experience

* Each browser must natively support a transaction confirmation user experience.
* Although we anticipate that in most cases the browser will render the transaction confirmation user experience, the protocol must support rendering by other entities (e.g., the operating system or authenticator).
* For regulatory reasons, the party that invokes SPC must be able to specify a timeout for the user experience. See [issue 67](https://github.com/w3c/secure-payment-confirmation/issues/67).
* The transaction confirmation user experience should include the beneficiary name, and optionally the title and favicon of the page where it was called. See [issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48).

### Sources of Instrument Information

* The protocol should support multiple ways of accessing the instrument display information, including browser storage, authenticator storage, and merchant-provided data. Note that merchant data may not be trustworthy, but during subsequent
authorization it can be validated against relying-party stored instrument display data.

### Cross-Browser Support

* If the user enrolled an SPC Credential when using one instance of a browser, it should be possible to leverage that authentication from a different instance of the same browser (e.g., both browsers are Firefox)
* If the user enrolled an SPC Credential when using one instance of a browser, it should be possible to leverage that authentication from any browser (e.g., one browser is Firefox and the other is Chrome).

Note: These two will conflict fairly directly with 'Sources of Instrument Information' above - if information is stored in the browser it will be by default not cross-browser (not even cross-instance of browser possibly). That might be fine, we might just say "ok you need to do merchant-provided data then".

### Low Friction Flows

* The browser should support transaction confirmation without hardware
  authentication (e.g., no FIDO user presence check) when requested by
  the relying party.

Q: Is the goal here to still produce a cryptogram from the authenticator? Or is it just being able to say 'well, I called the SPC API and the user clicked yes', with no actual 'proof' of that for anyone?

Overall, I'm not sure how we'll view this from the browser side, mostly due to lack of knowledge/experience on my part. I can only go from the fact that WebAuthn doesn't support user-not-present I believe, and there is assumedly a reason they did that (albeit that may change in the future!).

* For each transaction, a merchant should be able to express to the relying party a preference for a low-friction flow (or not to use a low-friction flow).

* If the browser supports a low-friction flow option, the browser must
  support a user preference to override that option and maintain the
  full hardware-supported (e.g., biometric) flow.

### Zero Friction Flows

* TBD

## Unenrollment

* The ecosystem should enable the user to communicate to the relying party to forget SPC Credential Identifiers. This might happen in a variety of ways (e.g., forget this authenticator and all associated instruments; forget any authenticators associated with this instrument, etc.). See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63).

## SPC Credentials

* When more than one SPC Credential matches input data, the browser must choose the authenticator in the order of the input SPC Credential Identifiers.
* When no SPC Credential matches input data, the protocol should terminate without any user experience to allow for seamless fallback behaviors.

Note: Whilst I'm not saying this will or won't be a challenge for SPC, please note this does violate WebAuthn's [Authentication Ceremony Privacy](https://www.w3.org/TR/webauthn-2/#sctn-assertion-privacy). The explainer does [recognize this today](https://github.com/w3c/secure-payment-confirmation/blob/gh-pages/README.md#probing), but I'm still exploring the ramifications on our side.

* If the protocol supports more than one instrument per authenticator (e.g., within the same SPC Credential), then each instrument must be uniquely addressable and have unique display information.

### Lifecycle Management

* The user must be able to remove individual SPC Credentials from a browser instance.

Note: My off-hand expectation here from the browser vendor side is that we will support users to remove the information we store in the browser profile, but we may not support removing individual credentials.

## SPC Assertions

* The SPC Assertion must include at least: a merchant origin/identifier, amount and currency, transaction id.
* The SPC Assertion must also include information about the user's journey. This information may be part of the authenticator's own assertion. For example, the assertion must indicate whether the user completed the transaction without a user presence check.
* The SPC Assertion must also include information about which entity displayed the transaction confirmation user experience (browser, OS, or authenticator).

Q: For the last two bullets above this (user journey and which entity), I'm not opposed but do want to understand why these two are desirable.

* The SPC Assertion must include a signature over that data (based on the associated authenticator). This signature may be validated by the Relying Party or any other party with the appropriate key.

## Security and Privacy Considerations

* For privacy, the protocol must enable the user to require that two payment instruments NOT be associated with the same authenticator.

* SPC Credential Identifiers must be origin-bound to reduce the risk
  of cross-site tracking through the protocol. This implies that the
  RP generates new SPC Credential Identifiers across merchants. The
  browser maps SPC Credential Identifiers to stored SPC Credentials.

Q: I'm very unclear on this requirement. It seems very far from everything we've discussed on SPC so far, and is introducing a redirection layer?

* It is not a requirement to obfuscate SPC Credential Identifiers used
  as input to SPC.

## FIDO Considerations

* FIDO credentials should be "enhanceable" to SPC Credentials.
* SPC credentials should also usable as ordinary FIDO credentials. See [issue 39](https://github.com/w3c/secure-payment-confirmation/issues/39).
* SPC credentials must be programmatically distinguishable from FIDO credentials.

Q: I'm not clear why this is necessary? (Not saying we shouldn't do it :D)

* SPC should support both local and roaming authenticators. See [issue 31](https://github.com/w3c/secure-payment-confirmation/issues/31) on discoverable credentials and [issue 12](https://github.com/w3c/secure-payment-confirmation/issues/12) on roaming authenticator behaviors.
* [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce enrollment costs. Use case: I enroll my authenticator via one browser, but stored data can be used in another browser.

## Detailed Considerations

Note: Ideally this level of detail would not be part of the scope
document. These musings are likely to migrate to a specification
once that becomes available.

### SPC Credential

* An SPC Credential is likely to include the following kind of data:
* One or more Payment Credential Identifiers. See [issue 13](https://github.com/w3c/secure-payment-confirmation/issues/13) on cardinality between SPC Credential and instruments). Also see [issue 27](https://github.com/w3c/secure-payment-confirmation/issues/27) about allowing user to request that each instrument have its own credential.
* Authentication-method specific data (e.g., rpid).
* See [issue 62](https://github.com/w3c/secure-payment-confirmation/issues/62) about associating a credential with a user profile. This issue discusses the idea of making that profile information available prior to instrument selection, which could support additional selection use cases.

### SPC Credential Identifiers
* See [issue 49](https://github.com/w3c/secure-payment-confirmation/issues/49) and [issue 10](https://github.com/w3c/secure-payment-confirmation/issues/10) on the nature of the Payment Credential Identifier.

### Sources of Randomness

Many authentication protocols include a source of randomness to ensure freshness. 

* See [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on any requirements for the nonce (e.g., random? secret?).
* See [issue 26](https://github.com/w3c/secure-payment-confirmation/issues/26) on not constraining the source of randomness

### SPC Assertion

* What is the nature of the signature? See [issue 40](https://github.com/w3c/secure-payment-confirmation/issues/40) and [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28)

### Security and Privacy Considerations

* See [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on security properties.
* See [issue 13](https://github.com/w3c/secure-payment-confirmation/issues/13) on cardinality between SPC Credential and instruments). Also see [issue 27](https://github.com/w3c/secure-payment-confirmation/issues/27) about allowing user to request that each instrument have its own credential.

## Editors

* Ian Jacobs

