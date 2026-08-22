# Nip for "Virtual/Hidden Relays"

This nip should introduce a way of communication with a "Virtual/Hidden Relay", both identified by a nostr-npub, over nostr events. 


The "Virtual/Hidden Relay" creates a nostr-nsec/npub identity, or is attributed with an existing one. 
It publishes a kind `10112`-event with relays / "rendez-vous points" on which it listens to requests and sends responses.

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the "Virtual/Hidden Relay">,
  "kind": 10112, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["r", <relay as defined in other Nostr protocol">],
    ...,
    ["r", <relay as defined in other Nostr protocol">]
  ],
  "content": "",
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

The "Virtual/Hidden Relay" publishes a relay information event, in the following structure.

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the "Virtual/Hidden Relay">,
  "kind": 10113, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["encryption", <definition of supported encryption, compare e.g. use in NIP47 info event, e.g. "nip44_v2">]
  ],
  "content": "<NIP-11 relay Information Document>",
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

For display and adressing there should be a new bech32 string-format in the format 'nrv1...'. The bech32-string should follow in general the instructions of NIP19. The prefix should be 'nrv'. TLV 0 is the 32 bytes of the pubkey key of the "hidden relay". TLV 1 is the specified rende-vous(rv)-relays of the "hidden relay".
"Virtual/Hidden Relay" are referenced in events requiring an r with a tag format of ["r", "nrv1..."].

The nrv-"Virthual/Hidden Relay" communication event has the following pattern:

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the sender of the message (e.g. "Virtual/Hidden Relay" or other party>,
  "kind": 27901, // as defined in NIP-01 an ephemeral kind-number is used for this event-type
  "tags": [
    ["p", <32-bytes lowercase hex-encoded public key of the receiver of the message (e.g. "Virtual/Hidden Relay" or other party>],
    ["encryption", <definition of encryption, e.g. "nip44_v2">]
  ],
  "content": encrypt_as_defined_in_encryption_tag({
    [<messages between client and relay as defined in other nostr-protocol>, ...]
  }),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

The messages in the encrypted content are ordered in an array, so it is possible to send multiple messages in one kind 27901-event. 
The content should be limited to some size so that it is transferable by most relays.

The nrv communication events should be treated as ephemeral by relays.
