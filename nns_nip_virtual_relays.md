# Nostr network stack (nns) nip - "Virtual/Hidden Relays"

This nip should introduce a way of communication between a "Virtual Client"/user with a "Virtual/Hidden Relay", both identified by a nostr-npub, over nostr events. 


The "Virtual/Hidden Relay" creates a nostr-nsec/npub identity, or is attributed with an existing one. It publishes a kind `10112`-event with relays / "rendez-vous points" on which it listens to nns-requests and sends nns-responses.
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

The "Virtual/Hidden Relay" publishes, or delivers on request, a nostr-info event, in the following structure:

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

The user of the Virtual/Hidden Relay references it in events requiring an r-tag (e.g. kind 10002) with ["r", "nns://pubkey", "<rendez-vous-relay1>", "<rendez-vous-relay2>", ...]. Clients detecting the nns:// prefix treat the tag as a hidden/virtual relay reference and use the pubkey and rendezvous relays.
For display there should be a new bech32 string-format in the format 'nrvrelay1...'. The bech32-string should follow in general the instructions of NIP19. The prefix should be 'nrvrelay'. TLV 0 is the 32 bytes of the pubkey key of the "hidden relay". TLV 1 is the specified rendez-vous(rv)-relays of the "hidden relay". TLV 2 is the optional pubkey of the administrator.

The nns communication event has the following pattern:

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the sender of the message (e.g. "Virtual/Hidden Relay" or "Virtual Client">,
  "kind": 27901, // as defined in NIP-01 an ephemeral kind-number is used for this event-type
  "tags": [
    ["p", <32-bytes lowercase hex-encoded public key of the receiver of the message (e.g. "Virtual/Hidden Relay" or "Virtual Client"/user>],
    ["encryption", <definition of encryption, e.g. "nip44_v2">]
  ],
  "content": encrypt_as_defined_in_encryption_tag({
    <messages between client and relay as defined in other nostr-protocol>
  }),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

The nns communication events should be treated as ephemeral by relays.
