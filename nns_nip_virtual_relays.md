# Nostr network stack (nns) nip - "Virtual/Hidden Relays"

This nip should introduce a way of communication between a "Virtual Client" with a "Virtual/Hidden Relay", both identified by a nostr-npub, over nostr events. 


The "Virtual/Hidden Relay" creates a nostr-nsec/npub identity, or is attributed with an existing one. It publishes a kind `10112`-event with relays on which it listens to nns-requests and sends nns-responses.
The kind `10112`-event follows the structure of NIP65 and NIP51 and signals that the pukey is reachable for nns-communication there.

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the "Virtual/Hidden Relay">,
  "kind": 10112, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["r", <relay as defined in other Nostr protocol">]
  ],
  "content": "",
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~


The user of the "Virtual/Hidden Relay" references the "Virtual Relay" in events that need a `r`-tag (e.g. kind `10002`), with ["r", "nns://npub1..."], or alternatively with ["r", "nns://nprofile1..."] (to include a relay hint directly, where to find the 10112 and 10113 events).


The "Virtual/Hidden Relay" publishes a nostr-info event, in the following structure:

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the "Virtual/Hidden Relay">,
  "kind": 10113, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["encryption", <definition of supported encryption, compare e.g. use in NIP47 info event, e.g. "nip44_v2 nip04 ...">]
  ],
  "content": "",
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~


The nns communication event has the following pattern:

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the sender of the message (e.g. "Virtual/Hidden Relay" or "Virtual Client">,
  "kind": 27901, // as defined in NIP-01 an ephemeral kind-number is used for this event-type
  "tags": [
    ["p", <32-bytes lowercase hex-encoded public key of the receiver of the message (e.g. "Virtual/Hidden Relay" or "Virtual Client">],
    ["encryption", <definition of encryption, compare e.g. use in NIP47, e.g. "nip44_v2">]
  ],
  "content": encrypt_as_defined_in_encryption_tag({
    <messages between client and relay as defined in e.g. NIP01, NIP77, ... in the nostr-protocol>
  }),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

The nns communication events should be treated as ephemeral by relays.
