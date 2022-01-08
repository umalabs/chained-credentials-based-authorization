# UMA Macaroons

## Tagline

Macaroons, made from scratch using an UMA recipe with the fresh HMAC ingredients.

## Introduction

Bearer tokens are vulnerable at rest and in transit when an attacker is able to intercept a token to illegally access private information. In order to mitigate some of the risk associated with bearer tokens, UMA Macaroons may be used instead of bearer tokens. UMA Macaroons are cryptographically chained blocks of data bearing a chronological tamper-resistant records of all their possessors and the changes that have been made to them. In the authorization flow, the UMA Macaroons ecosystem relies on auditability of records rather than on token minting and verification. UMA Macaroons adopt the User-Managed Access concept of authorization server, resource server, client, resource owner and requesting party.

## Key Differences from Google Macaroons

* Authenticated possessors.
* Claims are used instead of caveats.
* Different HMAC chaining.
* Verification at the authorization server.

Following we use the term *macaroon* to refer to UMA Macaroon.

## Concept of MACs Chaining

To create Macaroons we use the HMAC chaining construct *MAC*&#160;=&#160;HMAC(*K*,&#160;HMAC(*MAC*,&#160;*m*)) broken down into individual MACs,

*MAC*&#160;=&#160;HMAC(*MAC*,&#160;*m*)<br>
*MAC*&#160;=&#160;HMAC(*K*,&#160;*MAC*)

which forms the basis of the Macaroons chaining mechanism.

To simplify notation, we use the Double HMAC construct – a nested HMAC function, denoted by DHMAC, that takes 3 inputs (*K*,&#160;*MAC*,&#160;*m*) and outputs a message authentication code

*MAC*&#160;=&#160;DHMAC(*K*,&#160;*MAC*,&#160;*m*)&#160;=&#160;HMAC(*K*,&#160;HMAC(*MAC*,&#160;*m*))

where *K* is the secret key, *MAC* is the input message authentication code, and *m* is the message to be authenticated.

Macaroons possessors must be registered at the authorization server (public clients can use dynamic registration to become confidential clients). Macaroons are verified via the introspection endpoint of the authorization server.

## Use Cases

Advanced authorization scenarios e.g. chained resource servers.

### Example of Chained Macaroons

Each macaroon contains three mandatory claims:

* The random NONCE to prevent replay attack.
* The timestamp of when the macaroon was created.
* The URI that identifies who created the macaroon.

Additional groups of optional claims (e.g. in JSON format) can be added at any time until the macaroon is sent to the next possessor.

The HMAC chain may started with an AS or any other registered client. 

- The AS is the first macaroon possessor.<br><br>
*MAC*<sub>*AS*</sub> = HMAC(*K*<sub>*AS*</sub>, NONCE<sub>*AS*</sub>)<br>
*MAC*<sub>*AS*</sub> = DHMAC(*K*<sub>*AS*</sub>, *MAC*<sub>*AS*</sub>, Timestamp<sub>*AS*</sub>)<br>
*MAC*<sub>*AS*</sub> = DHMAC(*K*<sub>*AS*</sub>, *MAC*<sub>*AS*</sub>, URI<sub>*AS*</sub>)<br>
*MAC*<sub>*AS*</sub> = DHMAC(*K*<sub>*AS*</sub>, *MAC*<sub>*AS*</sub>, *claims_1*<sub>*AS*</sub>)<br>
...<br>
*MAC*<sub>*AS*</sub> = DHMAC(*K*<sub>*AS*</sub>, *MAC*<sub>*AS*</sub>, *claims_n*<sub>*AS*</sub>)<br>
*MAC*<sub>*AS*</sub> = HMAC(*K*<sub>*AS*</sub>, *MAC*<sub>*AS*</sub>)<br><br>
- Hop to the next possessor – the client.<br><br>
*MAC*<sub>*client*</sub> = HMAC(*K*<sub>*client*</sub>, NONCE<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, Timestamp<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, URI<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *MAC*<sub>*AS*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *claims_1*<sub>*client*</sub>)<br>
...<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *claims_n*<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = HMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>)<br><br>
- Hop to the next possessor – the RS_1.<br><br>
*MAC*<sub>*RS_1*</sub> = HMAC(*K*<sub>*RS_1*</sub>, NONCE<sub>*RS_1*</sub>)<br>
*MAC*<sub>*RS_1*</sub> = DHMAC(*K*<sub>*RS_1*</sub>, *MAC*<sub>*RS_1*</sub>, Timestamp<sub>*RS_1*</sub>)<br>
*MAC*<sub>*RS_1*</sub> = DHMAC(*K*<sub>*RS_1*</sub>, *MAC*<sub>*RS_1*</sub>, URI<sub>*RS_1*</sub>)<br>
*MAC*<sub>*RS_1*</sub> = DHMAC(*K*<sub>*RS_1*</sub>, *MAC*<sub>*RS_1*</sub>, *MAC*<sub>*client*</sub>)<br>
*MAC*<sub>*RS_1*</sub> = DHMAC(*K*<sub>*RS_1*</sub>, *MAC*<sub>*RS_1*</sub>, *claims_1*<sub>*RS_1*</sub>)<br>
...<br>
*MAC*<sub>*RS_1*</sub> = DHMAC(*K*<sub>*RS_1*</sub>, *MAC*<sub>*RS_1*</sub>, *claims_n*<sub>*RS_1*</sub>)<br>
*MAC*<sub>*RS_1*</sub> = HMAC(*K*<sub>*RS_1*</sub>, *MAC*<sub>*RS_1*</sub>)<br><br>
- Hop to the next possessor – the RS_2.<br><br>
*MAC*<sub>*RS_2*</sub> = HMAC(*K*<sub>*RS_2*</sub>, NONCE<sub>*RS_2*</sub>)<br>
*MAC*<sub>*RS_2*</sub> = DHMAC(*K*<sub>*RS_2*</sub>, *MAC*<sub>*RS_2*</sub>, Timestamp<sub>*RS_2*</sub>)<br>
*MAC*<sub>*RS_2*</sub> = DHMAC(*K*<sub>*RS_2*</sub>, *MAC*<sub>*RS_2*</sub>, URI<sub>*RS_2*</sub>)<br>
*MAC*<sub>*RS_2*</sub> = DHMAC(*K*<sub>*RS_2*</sub>, *MAC*<sub>*RS_2*</sub>, *MAC*<sub>*RS_1*</sub>)<br>
*MAC*<sub>*RS_2*</sub> = DHMAC(K<sub>*RS_2*</sub>, *MAC*<sub>*RS_2*</sub>, *claims_1*<sub>*RS_2*</sub>)<br>
...<br>
*MAC*<sub>*RS_2*</sub> = DHMAC(*K*<sub>*RS_2*</sub>, *MAC*<sub>*RS_2*</sub>, *claims_2*<sub>*RS_2*</sub>)<br>
*MAC*<sub>*RS_2*</sub> = HMAC(*K*<sub>*RS_2*</sub>, *MAC*<sub>*RS_2*</sub>)<br>

The last *MAC*<sub>*RS_2*</sub> can be verified via the introspection endpoint of the AS.

## Nested Macaroon / Third-Party Claims

A macaroon can contain another macaroon.

### Example of Nested Macaroon

This is an excerpt from the above Example of Chained Macaroons extended by third party claims.

...
- Hop to the next possessor – the client.<br><br>
*MAC*<sub>*client*</sub> = HMAC(*K*<sub>*client*</sub>, NONCE<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, Timestamp<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, URI<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *MAC*<sub>*AS*</sub>)<br><br>
- Hop to the next possessor – the AS_third_party.<br><br>
*MAC*<sub>*AS_third_party*</sub> = HMAC(*K*<sub>*AS_third_party*</sub>, NONCE<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, Timestamp<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, URI<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, *MAC*<sub>*client*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, *claims_1*<sub>*AS_third_party*</sub>)<br>
...<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, *claims_n*<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = HMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>)<br><br>
- Hop to the next possessor – back to the client.<br><br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *MAC*<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *claims_1*<sub>*client*</sub>)<br>
...<br>
*MAC*<sub>*client*</sub> = DHMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>, *claims_n*<sub>*client*</sub>)<br>
*MAC*<sub>*client*</sub> = HMAC(*K*<sub>*client*</sub>, *MAC*<sub>*client*</sub>)<br><br>
- Hop to the next possessor – the RS_1.

...

## Confidential Claims

~~Third-party claims can be chained using the AES-GCM authenticated encryption algorithm instead of the HMAC message authentication algorithm.~~

This is an excerpt from the above Example of Nested Macaroon extended by confidential third party claims.

...
- Hop to the next possessor – the AS_third_party.<br><br>
*MAC*<sub>*AS_third_party*</sub> = HMAC(*K*<sub>*AS_third_party*</sub>, NONCE<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, Timestamp<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, URI<sub>*AS_third_party*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, *MAC*<sub>*client*</sub>)<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, Enc(*K*<sub>*AS_third_party*</sub>, *claims_1*<sub>*AS_third_party*</sub>))<br>
...<br>
*MAC*<sub>*AS_third_party*</sub> = DHMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>, Enc(*K*<sub>*AS_third_party*</sub>, *claims_n*<sub>*AS_third_party*</sub>))<br>
*MAC*<sub>*AS_third_party*</sub> = HMAC(*K*<sub>*AS_third_party*</sub>, *MAC*<sub>*AS_third_party*</sub>)<br><br>
- Hop to the next possessor – back to the client.

...

## Conclusion

(TBD)

## Acknowledgment

Credits go to [WG - User-Managed Access][1] and [Google Research Publications][2].

[1]: https://kantarainitiative.org/confluence/display/uma/Home
[2]: https://research.google/pubs/pub41892/
[3]: https://github.com/umalabs/uma-pocop-tokens
[4]: https://github.com/umalabs/uma-pocop-tokens#chained-macs-with-multiple-messages
[5]: https://github.com/umalabs/uma-pocop-tokens#chained-macs-with-multiple-keys
[6]: https://github.com/umalabs/uma-pocop-tokens#pocop-token-mechanism
