KMS to SDN API for QKDN<!-- omit in toc -->
==

- [1. Overview](#1-overview)
- [2. Implementation specifics](#2-implementation-specifics)
- [3. Notes](#3-notes)
  - [3.1. Scope](#31-scope)
  - [3.2. Security](#32-security)
  - [3.3. ETSI GS QKD 015 compatibility](#33-etsi-gs-qkd-015-compatibility)
  - [3.4. Related publications](#34-related-publications)
- [4. Acknowledgements](#4-acknowledgements)

This repository hosts and maintains the API description developed by AIT for an interface between a Key Management System (KMS) and a Software Defined Network (SDN) Agent for Quantum Key Distribution Networks (QKDN).

# 1. Overview

![SDN Managed QKDN](doc/figures/ETSI_015_SDN_Network.png)

The figure above depicts a common way of deploying an SDN managed QKD Network according to ETSI GS QKD 015. The interfaces of such a network are marked as follows:

a) KMS to Application Interface, typically ETSI GS QKD [014](https://www.etsi.org/deliver/etsi_gs/QKD/001_099/014/01.01.01_60/gs_qkd014v010101p.pdf) or [004](https://www.etsi.org/deliver/etsi_gs/QKD/001_099/004/02.01.01_60/gs_qkd004v020101p.pdf).

b) KMS to QKD interface, typically ETSI GS QKD [014](https://www.etsi.org/deliver/etsi_gs/QKD/001_099/014/01.01.01_60/gs_qkd014v010101p.pdf) or [004](https://www.etsi.org/deliver/etsi_gs/QKD/001_099/004/02.01.01_60/gs_qkd004v020101p.pdf).

c) KMS to SDN Agent interface.

d) SDN Agent to QKD interface, typically ETSI GS QKD [023 (draft)](https://portal.etsi.org/webapp/WorkProgram/Report_WorkItem.asp?WKI_ID=69537).

e) SDN Agent to SDN Controller Interface, typically ETSI GS QKD [015](https://www.etsi.org/deliver/etsi_gs/QKD/001_099/015/02.01.01_60/gs_QKD015v020101p.pdf).

q) Quantum channel, typically optical fiber.

As it becomes apparent from this list, the KMS to SDN-Agent interface lacks clear specification by ETSI GS QKD or any other organization. Therefore, AIT developed a simple API with the goal of being simplistic enough to have a low barrier for adaption and provide a feature set, which satisfies the requirements of an SDN managed QKDN.

An earlier version of this API was developed together with Universidad Politécnica de Madrid (UPM), Nextworks and Telefonica within the scope of the [DISCRETION project](https://discretion-eu.com/).

# 2. Implementation specifics

<img src="doc/figures/API_components.png" height="400">

The API is implemented as a https REST API in a bidirectional setup, where the KMS and SDN Agent host a server and can act as a client to the other peer. Therefore, two APIs are described. The API further must support both, the ETSI 014 and ETSI 004 Application interface.

You can find the openAPI descriptions at:

- [`openapi/kms_to_sdn_api.yaml`](openapi/kms_to_sdn_api.yaml) for the API, where the SDN Agent hosts the server and the KMS initiates client requests.
- [`openapi/sdn_to_kms_api.yaml`](openapi/sdn_to_kms_api.yaml) for the API, where the KMS hosts the server and the SDN Agent initiates client requests.

Sequences and further information can be found in [`doc/readme.md`](doc/).

# 3. Notes

Some notes are given in this section.

## 3.1. Scope

This API's scope is limited to the interaction between the KMS and SDN Agent in the context of QKD Networks. Explicitly beyond scope are any details on the SDN Controller, QKD Layer, key establishment in QKD Networks or cryptographic aspects.

## 3.2. Security

The SDN Agent and KMS are supposed to be deployed in the same security perimeter, also referred to as trusted node. Attacks on the API should therefore be prevented by the perimeter security mechanisms. However, as the technical implementation hurdles are very low, TLS 1.3 is required for this interface.

## 3.3. ETSI GS QKD 015 compatibility

Unfortunately some bugs in the ETSI GS QKD 015 v2.1.1. specification are not resolved yet. If strictly following ETSI GS QDK 015 some issues will arise:

- ETSI GS QKD 015 v2.1.1. specifies that the SDN Controller first must be notified from both endpoints before a relay path is established. This may be possible with ETSI GS QKD 004, but is incompatible with ETSI GS QKD 014 (the more adopted specification). ETSI GS QKD 014 clearly already needs the final keys at `enc_keys` before the second node even knows of the `dec_keys` request. Therefore the path must be established at the first request at the source.
- ETSI GS QKD 015 v2.1.1. does not publish the most important metric, which is the key availability (KAV) at the KMS layer. This API publishes it as `KAV` via the `link/performace/{link_id}` endpoint, but if required the SDN Controller can derive the ESKR: $ESKR = \dfrac{\Delta KAV}{\Delta t}$

## 3.4. Related publications

This repository complements research presented in the following publications:

- James, P, Laschet, S, Ramacher, S & Torresetti, L 2023, Key Management Systems for Large-Scale Quantum Key Distribution Networks. in ARES '23: Proceedings of the 18th International Conference on Availability, Reliability and Security., 126, ACM International Conference Proceeding Series, S. 1-9, ARES 2023: The 18th International Conference on Availability, Reliability and Security, Benevento, Italien, 29/08/23. https://doi.org/10.1145/3600160.3605050
- Brito, JP, Ballesta, J, Brito-Mendez, R, Mengual, L, Ortíz, L, Martin, V, Cantó, R, Muñiz, A, Pastor, A, Lopez, D, Laschet, S, Ramacher, S, Piscione, P, Abdulwahed, AK, Giardina, P, Freitas, M, Calé, R, Maia, L, Magalhães, L, Anjos, G, Chaves, R, Afonso, J, Martins, P, Dias, T, Pinto, F, Vieira, M, Bacar, R & Bastos, C 2025, Secure Network Innovation in Defense: SDN and Quantum Cryptography with DISCRETION. in 2025 International Conference on Quantum Communications, Networking, and Computing (QCNC). S. 261 - 268, International Conference on Quantum Communications, Networking, and Computing (QCNC 2025), Nara, Japan, 31/03/25. https://doi.org/10.1109/QCNC64685.2025.00049
- Valbusa, F, Lorünser, T, Spini, G & Laschet, S 2025, Relaxing the Single Point of Failure in Quantum Key Distribution Networks: An Overview of Multi-path Approaches. in F Skopik, V Naessens & B De Sutter (Hrsg.), Availability, Reliability and Security: ARES 2025 EU Projects Symposium Workshops, Ghent, Belgium, August 11–14, 2025, Proceedings, Part I. Bd. 15998, Lecture Notes in Computer Science, Bd. 15998, Springer, S. 183–200, ARES 2025 EU Projects Symposium Workshops, Ghent, Belgien, 11/08/25. https://doi.org/10.1007/978-3-032-00642-4_11
- Bastos, C, Pinto, F, Bacar, R, Anjos, G, Almeida, M, Pinto, AN, Chaves, R, Dias, T, Afonso, J, Calé, R, Freitas, M, Maia, L, Magalhães, L, Muñiz, A, Cantó, R, Brito, JP, Ballesta, J, Méndez, RB, Laschet, S, Ramacher, S, James, P, Torresetti, L, Piscione, P, Abdulwahed, AK, Giardina, P, Martin, V, Ortiz, L, Pastor, A, Muga, N, Silva, N, López, D, Vieira, M, Escribano, C & Mengal, L 2025, DISCRETION: First Field Demonstration of a Quantum Enabled SDN in the Context of a Military Exercise. in 2025 International Conference on Military Communication and Information Systems (ICMCIS). S. 11-18, 2025 International Conference on Military Communication and Information Systems (ICMCIS), Oeiras, Portugal, 13/05/25. https://doi.org/10.1109/icmcis64378.2025.11047713

# 4. Acknowledgements

Different aspects of this work were enabled by Co-funding:

From Digital Europe Program under project numbers 101091642 ("QCI-CAT"), 101091588 ("QUARTER"), and 101091564 ("eCausis").
From European Union’s Horizon Europe research and innovation program under Grant Agreement No.~101114043 ("QSNP").
From the Österreichische Forschungsförderungsgesellschaft mbH (FFG) research program "Breitband Austria 2030: GigaApp 2. Ausschreibung" under Project Number: FO999917949 ("Q-Crit Austria").

![EU co-funding logo](https://www.eacea.ec.europa.eu/sites/default/files/styles/embed_large_2x/public/2022-11/EN%20Co-Funded%20by%20the%20EU_POS.png)

![FFG co-funding logo](doc/figures/FFG_Logo_EN_RGB_1500px.png)
