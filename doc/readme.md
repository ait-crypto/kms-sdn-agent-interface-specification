Specification details<!-- omit in toc -->
==

- [1. Introduction](#1-introduction)
- [2. Sequences](#2-sequences)
  - [2.1. Link establishment](#21-link-establishment)
  - [2.2. Link update (ETSI 004 use-case)](#22-link-update-etsi-004-use-case)
  - [2.3. Key stream close (ETSI 004 use-case)](#23-key-stream-close-etsi-004-use-case)

# 1. Introduction

This document gives some additional details beyond the high level picture outlined in the top level [`readme.md`](../readme.md)

# 2. Sequences

The following section states some common sequences for different use-cases and how this API is supposed to support the process. Some aspects, which are beyond this scope, are shortened.

In the sequences, the bold and green highlighted messages are within the scope of this API specification, all others are beyond the scope and only given for context. The data transmitted is also only a hint and does not display all data of that message.

## 2.1. Link establishment

As an application either opens a key stream via the ETSI GS QKD 004 `open_connect` or requests keys via ETSI GS QKD 014 `enc_keys`, the KMS first has to know via which path through the network the key(s) should be established. The error-free process works as follows and is displayed in a sequence diagram thereafter (error handling not described here):

- The application starts the request at the source KMS (msg 1, 2)
- The source KMS notifies its SDN Agent of the request (msg 3)
- The SDN Agent in turn notifies the SDN Controller (msg 4), which computes the path (msg 5) and notifies all SDN Agents in the relay path (msg 6, 8).
- The SDN Agents on the relay path configure the path with the next node infos to their corresponding KMS instances (msg 7, 9)
- The original source KMS message, after successful establishment of the relay path, is answered by the SDN Controller to the SDN Agent (msg 10) and in turn to the source KMS (msg 11)
- If the request was an `open_connect` via the ETSI GS QKD 004 interface, it is answered (msg 12) and as soon as the destination application requests the same key stream to be opened (msg 13, 14) the KMS notifies the SDN Agent of this link update (msg 15).
- If the request was an `enc_keys` via the ETSI GS QKD 014 interface, first the keys have to be established using the link (msg 18, 19, 20) and upon success the request can be answered (msg 21). Then the destination App (msg 22) requests the same keys via the `dec_keys` endpoint (msg 23). The KMS informs the SDN agent of this link update (msg 24) and delivers the keys (msg 26)

![sequence](figures/sequence_sdn_new_app.png)

**Note 1:**
In case of the ETSI GS QKD 004, the established relay path can be used for all subsequent key requests (stateful).

**Note 2:**
In case of the ETSI GS QKD 014, each request by the application results in a new SDN request (stateless).

**Note 3:**
The KMS should request from the SDN paths which can also support any internal key consumption.

**Note 4:**
All communication beyond scope is only exemplary, specifically the communication between the SDN Controller and SDN Agent (messages 4, 6, 8, 10, 16, 25) is out of scope.

## 2.2. Link update (ETSI 004 use-case)

Since the ETSI GS QKD 004 is stateful, it can maintain its path. But high key consumption on a link or an unexpected event (e.g. QKD node unreachable) requires an update of the established path. The error-free process works as follows and is displayed in a sequence diagram thereafter (error handling not described here):

- The SDN Controller for some reason (e.g., load balancing event) recalculates the relay path (msg 5) and informs all SDN Agents of the current path (msg 6, 8, 12) and the new node SDN Agent (msg 10) of the change.
- The SDN Agents update (msg 7, 13), newly install (msg 11) or delete (msg 9) the KMS relay configuration, correspondingly if parts of the data are updated, completely new or to be removed.

This update works seamlessly without the application layer noticing, since normal operation (msg 1-4) continues after the newly established relay path (msg 17-20).

![path update](figures/sequence_sdn_update_path.png)

**Note 1:**

This message can also be used to tear down a link entirely by sending a `DELETE` to `/link/relay/{key_stream_id}` to every node, for example in case of a key stream close initiated by the SDN Controller.

**Note 2:**
All communication beyond scope is only exemplary, specifically the communication between the SDN Controller and SDN Agent (messages 6, 8, 10, 12) is out of scope.

## 2.3. Key stream close (ETSI 004 use-case)

The key stream of the ETSI GS QKD 004 can be closed by applications. The error-free process works as follows and is displayed in a sequence diagram thereafter (error handling not described here):

- After normal operation (msg 1-4), the application can close a session with the `close` message (msg 5).
- In this case the KMS notifies the SDN Agent (msg 6) of this event.
- In turn the Controller notifies all other SDN Agents (msg 8, 10)
- The SDN Agents notify the KMS (msg 9, 11), which delete their forwarding info and other relevant data associated with the key stream. The destination KMS must keep any keys for the negotiated TTL, for which this is the starting time (or until a close is issued (msg 16)).
- The response to the original SDN notification is issued (msg 13) and the application is informed of the successful closing of the key stream. The application then can inform its peer app (msg 15), so it can also close the key stream (msg 16, 17)

![path update](figures/sequence_sdn_etsi004_close.png)
