# Intro - Chris Wood

Compared TLS development to QUIC Development.
Described what motivated QUIPS.

# A security model & verified implementation of the QUIC record layer - Antoine Delignat-Lavaud

What is QUIC?
Motivations for QUIC.
How it works:
- Internal modularity
  - Research area: Use of TLS Traffic secrets being used directly by QUIC record layer
- Packet format
Open security problems in QUIC (at least 7):
  - New custom construction for encryption packets (focus of talk)
Verification Goals:
- Functional correctness
- Cryptographic security
- Verified implementation
QUIC record layer description
QUIC packet encryption
Modelling single packet encryption
An attack on pre-draft 13 packet encryption
Digression: are CIDs authenticated?
Specifying QUIC packets formats (everparse)
- New bitfield and bitsum combinations
Theorem: QUIC header format
Theorem: QUIC header encryption format
Decryption window condition
Going back to Security Model
Packet stream security definition
Proof: Code reduction
Remaining issues with QPE
- Some implementations skip the payload decryption based on the value of the decrypted packet number (this reveals information)
- Very tricky to implement constant-time decryption: first decrypt 2 lsb of flags, then truncate mask, then decrypt PN … unsafe
- Authentication of LN still depends on header formatting ,we prefer to include Ln in nonce (2msb are not used)
A simplified construction (propose provably secure nonce-hiding constructions to CFRG).
Evaluation
Conclusions

Questions:
CW: You mention constant time is tricky: is it constant time?
- Yes.
CW: What about the processing time of packet once decrypted? Does padding cause problems because it is linear?
- It is a concern.  You could do it, but it might be less flexible.  Sparse padding validation you need to consider the amount of frames that are included.
MT: ACK, STREAM, etc. processing are all different. When do you stop trying to be constant time?
- Unclear, since traffic analysis is hard and application specific.
- Might be worth keeping padding at the end?
CW: Is there any chance we can include CID length into version?
- Commitments have been made to note change things that are version specific.
- Expect applications to do their own version for variable length CID.
- Authenticating CID is, however, probably doable.  Need transport parameter for dcid.

# Two-Tier Authenticated Encryption: Nonce Hiding in QUIC - Felix Günther
What is the cryptographic primitive in hiding noce
QPE (Quick Packet Encryption)
Two-tier encryption: mixing authenticated encryption and malleable encryption.
Why a dedicated primitive?
Recap: Classical Nonce-based AE
Nonce-hdiing AE
Two-tier AE (it’s still one function on encryption, two-step description though)
You don’t specify the security properties!?
Yes this is just the syntactic party will get to security next.
Two-tier AE security
QPE Core: AEX (AES with XOR)
Doesn’t deal with varint
QPE is (partially nonce-hiding)?
Mapping two-tier AEX to  QPE
Future work:
Want to hide header bits
FS through rotating AE encryption
ADL (Antoine Delignat-Lavaud): The QUIC should looks at this because we could have a more flexible model
MT (Martin Thomson): Tier 2 is constant because we have them under the same protection
ADL: We don’t update header protection keys.
AH (Amir Hertzberg)): If you don’t update it - it sounds like it’s not actually one primitive?  The properties should hold.
FG (Felix Günther): We believe that there is benefit to keeping them together in one.  Allows us to model each tier separately and interactions between the two.
Further instantiations of two-tier AE
Application to other settings
Summary

Questions:
CW: What is the status of the CFRG draft?
ADL: Just an idea now.
ADL: It would be better to have the CFRG provably secure construct and try to get them adopted so that other WGs could just use it.
MT: We started out not shooting for much security.
CW (Chris Wood): Timing?
MT: Probably not going to line up, but once it’s done sure we could point to it.
CW: Why did we hide the key phase bit?
MT: Comes down to the correlation.
BT (Brian Trammel): Not much analysis about the marginal risk.
MT: If we could get further along with the analysis - if the one bit doesn’t make much difference then we could put it back in the clear.
BT: Does this generalize to n-tier authentication encryption?
FG: Sure … Understand this might not be ready for QUIC.
BT: We put them key phase and packet number bits because it was convenient.
ST (Sean Turner): TIme frame for CFRG?
Everybody: Who knows.
MT: Once we get into multiple tiers we have to think about things like computational costs.  Getting them to swallow one was too much more might be a bridge too far. 

# QuicR: QUIC Resiliency to BW-DoS Attacks - Amir Herzberg
BW = Bandwifth

BW-DDoS attacks cause Clodding: Loss and Latency
Defense against BW-DDoS, clogging
LE (Lars Eggert): duplicate packets (in a congested network) only works if not everybody does it!
What is QuicR is all about?
QuicR
Impact and Reaction to Clogging
Congestion-control responds dramatically to losses
QuicR’s modes of operation
QuicR’s FEC and Retransmission
ADL: In the case of QUIC where do you put the additional information?
AH: Send it as additional packets.
AH: Yes you would need to know the redundancy packets.
MB (Mike Boyle): Question about notation.
LE: Emulating DoDs by introducing random doS. This is not how it is usually done.  The loss rate you look at the rate of the DDoS not the queue.
AH: We have bombarded the queue previously.
Results
ADL: how many packets
AH: It’s about twice at 30%
Conclusion and To-Dos

Questions
If you want to deploy a FEC-based system to QUIC?
Need some additional information.
MT: I think you might need the information at the frame vs packet level.
Yes.
# Secure Communication Channel Establishment: TLS 1.3 (over TCP Fast Open) vs QUIC - Samuel Jero 
Secure Communication on the Web
How is Internet Traffic Encrypted
Why Low Latency is Important
Low-Latency Protocols
TLS 1.3/TFO
Provable Security Approach
Prior Work: TLS1.3 vs QUIC
Our Work: Comparing Layered Protocols
Step 1: Protocol Syntax
Step 2: Security Model
ADL: Do you prove KE Header and Payload Integrity?
SJ (Samuel Jero) We tried to prove that. And, no you don’t get any of that for these protocols.
ADL: Channel Security - what granularity level of authentication?
SJ: We went for a coarse grain model.
HM: Why couldn’t you just use the session level keys?
MT: I will get into that later.  The server might not have access to session level keys.
SJ: You could probably develop the new security property could include that.
Summary of Security Results
SJ: Nobody gets header and payload security. But no practical attack found against gQUIC/QUIC.
ADL: Paper proof?
SJ: Yes.
MT: Have you disclosed these?
SJ: Yes.
Reset Authentication
TFO Cookie Removal Attack
MT: Not deployed in FF.
BT: 2017 scans reported 3% and 83.9 where 1.3.5.6.9 (i.e., Google).
# Robust Channels: Handling Unreliable Network Messages in QUIC’s Record Layer - Felix Günther
Recap: Secure Channels over TCP
Handling Unreliable Transport
We’re not the first to look at channels
Generalizing Channel Correctness
FG: DTLS1.2 was claimed to be level 2.
MT: DTLS was not implemented like that.
AH: Reordering is within the window?
FG: Yes - reduces the amount of window.
MT: If you see a number far in the window - they will jump forward and then forget everything before.
ADL: If you have a dynamic scaling window then you need to change predicate?
FG: QUIC predicate needs the model specialized.
Defining (ROB) Robustness
INT (Integrity)
ROB-INT
Are there more constraints about an error not affecting later?
GH: You could count, but it’s not required.
A Robust HIerarchy
QPE
QUIC Channel
MT: Does this argue for more authentication a AEAD
GH: Gets to how you define primitives.
ADL: What it really means is that in QUIC you have to rekey more often.  You to need do it more quickly.
ADL: Does it make sense to count the number of errors at the recipient?  It seems to help you.
ADL: What you want to stop is a low recipient number.
MT: I may at a receiver count the number of packets thrown away and the correctly received packets and when it hits a limit rekey.
MT: No limit on the number of rekeys.

# QUIC Security Summary - Martin Thomson
QUIC handshake overview
ADL: The point of the handshake done is to get rid of the previous state.
MT: Yes and more on this later.
ADL: Another way to think about is that the on the client side - the clients know everything necessary to decrypt server messages.
ADL: In most cases, the TCP/SYN/ACK is done by the OS. Is it really that much worse to do token encryption?
MT: Because the spec doesn’t have any requirements about the token so it’s up to the application.
ADL: Initial versions was the QUIC could use TLS retry. Nice because you could save a roundtrip. Why did we move away?
MT: THere were people that wanted to do offload and didn’t want to deal with TLS-processing pipeline. You have to reconstruct those messages at the server.
BT: Wanted to do this token without needing the server secret.
ST: Bigger keys are going to cause lots of problems?
MT: Yes.
ADL: Omitting version negotiation is that it will always been an attack. It seems like transport parameters give you what you need.
MT: Not sure that was the form we wanted.
MB: You uncovered deadlocks.
MT: They came up in weird places.
Package and header protection
ADL: When can you start using CID.
MT: As soon as you start sending.
ADL: YOu cannot have a new CID frame inside a packet?
Key Update
Migration
Connection reset
AH: Do you also have any repeating of the token for the initial request/response.
It’s integrated in the token.
AH: If you can’t filter out reflection traffic?
Version negotiation and other stuff
# On Wire Images - Brian Trammel
What is a Wire Image and Why do I care?
The case of the Spin bit
Passive RTT estimation with TCP
Passive RTT estimation with the QUIC spin bit
Bidirectional RTT estimation with the QUIC spin bit
Spin bit and privacy
But it’s not quite so simple
Geolocation by exclusion
1ms RTT - 100km (+- a lot)
Location privacy recommendations
Probabilistic nonparticipation
ADL: What does it mean to run it?
BT: You run the algorithm.
Final thoughts
# Authenticated QUIC Handshakes - Chris Wood
A Normal Handshake
Packet Protection Keys
Initial Keys
Initial Authentication
Simple Mitigation
TLS Detour (Encrypted SNI)
Encrypted Client HellO (ECHO)
ECHO flow in TLS1.3
ECHO Mechanics
Back to QUIC:
Authenticated Handshake
ECHO-Based Proposal
Server-Side Processing
ECHO Authenticated Fallback
Downgrade Prevention
AH: DNS provides distribution and if you use DNSSEC authentication.
CW: It’s different because DNS is giving you the initial key then.
AH: You have a pk but it is not validated. You can either use DNSSEC or use the DoH.
CW: Not sure if servers would be up for that.
Authenticated Handshake (cont’d)
Repeat Yourself
Open Questions
# nQUIC (Noise+QUIC) - Alishah Chator
TLS Explanation
Handshake Modularity
Are there circumstances where we can do better?
What is Noise?
When might you not want to use TLS?
Peer Authentication and Pinning
nQUIC
nQUIC’s Noise Pattern
Why IK?
Anatomy of nQUIC
Hostname Section
Handshake Request/Response
Implicit Acknowledgement
Ratcheting in nQUIC
Interoperability
Cost Comparison
Performance Evaluation
Moving Forward
Conclusion
# Towards Securing the IoT with QUIC - Lars Eggert
QUIC on IoT devices
Warpcore and Quant
System hardware and software
Measurements - Code Size
Build size: 32-bit optimizations
Build size: client-only mode
Build size: server-only (didn’t do)
Build size: minimally-required crypto
Build size: no migration
Build size: no error reasons
Build size: no stateless resets
Build size; Drop re-ordered 0-RTT
Build size: Drop all reordered data
Build size: don’t maintain connection info
Down from 98 to ~60, but if you went hard at could get down to 30 KB (assumes access to hardware crypto)
Measurements - Stack and Heap
	Stack and heap usage
Stack usage: init phase
Stack usage: open phase
Stack usage: transfer phase
Stack usage: close phase
Heap usage: 
Measurement - Energy and Performance
	Energy
	Performance
Future work - MEasurements and Implementation

Questions:
ADL: Minimal overhead for code size memory overhead in TCP
LE: Yes it’s less for TCP.  I think it will be less but don’t know for sure.
BT: Show slides at QUIC WG.
LE: Talked to the T2T.
PANEL
CW: Lars what can the academic community do to help?
Lars: Nothing they can do to make it go faster. Would be valuable to raise any security concerns bring them up.

CW: Martin how would you sort the issues you identified?
MT: More concerned about other aspects like does the handshake terminate and or are there other places where we’d end up in deadlock.
ADL: The biggest threat to QUIC is that there is no version update. It seems like that should be high on the list of issues because we can’t patch any issues with a new version.
Lars: There is a design team to solve this with an extension.
MT: Very busy people on that DT.
Lars: Also need to care about implementations.  Look at Jana/Marteen’s testing rig.
CW: Verifiable implementations will help here.
ADL: If one exists then it proves it’s possible.
Lars: Bunch of implementations have been following but the cruft needs to get cut out.  Might be better to start from scratch.
CW: What would you say was the hardest part of the protocol to implement?
MT: 0-RTT was easy.  Hard in TLS but not so hard in QUIC.  And, this was a surprise.
ADL: 0-RTT is not a clear cut in QUIC.  But, this is a concern from me.  Nothing tells me that the data is 0-RTT - and so I don’t know what data was replayed.
Lars: It was all one big painspot.  It’s a lot of stuff.
ST: Squishing protocols together is going to necessarily mean more functions in the implementation.
Lars: It’s that and it’s complicated at the API level.
ADL: Need a new interface.  When you look at the implementations there are fewer TLS APIs.
FG: Finding the right abstraction is the challenge.  Should we have a seperate standard for APIs.
ADL: IETF doesn’t specify APIs
Lars; It kind of depends.
ADL: What would be a good way to make the modularity more can abstract away.  Could do this with a draft if you described in terms of a record layer.
MT: Not sure we captured all of the interactions.  There are a lot of subtle things. If you want to do a new handshake then it’s a new version.
ADL: It’s nice to have this division but for optimization we don’t require.
CW: Would it be useful to spec out what the handshake should look like?
MT: Not sure we need to that, but we do learn from them for the next time.

CW: Lots of discussion about what QUIC does more. CID still kind of leaks though.  We have some ways to protect against this leak.  In the past there has been discussions about encrypting the CID.  What’s the history and why did we land on this being the “key” to the connection.
MT: gQUIC didn’t have the concept that you could change the CID.  Along the way we were concerned about linkability.  It is an interesting party - and there are three parties involved and you.  You want to be able to construct things from the server that the load balancer needs to consume.  Client needs to construct them too.  Things just got complicated.
CW: Martin Duke’s draft starts to flush out the interactions needed.
MT: Yes, but it’s just the servers’ figuring it out not the client.
Lars; We want to encrypt more and more.  Are we past the point it’s useful?  Are there significant gains to be made?
ADL: Any privacy guarantee is not enough in practice because traffic security blows it all away.
CW: TLS1.3 considers it but does nothing.
ADL: Any additional construction to protect CID is broken by traffic flow security.
CW: Certainly things like fixed sized cells could be done easily.
MT: DNS padding turns out it wasn’t very helpful until you get to a certain point.
CW: Is there anything more we should be doing to help protect against traffic flow security?
MT: QUIC for TOR was to transport packets, but if an on-path attacker can force drops and check for retransmissions.
CW: We don’t need everything: low latency or bandwidth: which is more important for the common case?
SJ: It depends on what side you’re on.
MT: Server and clients both care about low latency.
BT: In the common case it’s not that big of a deal, but in the longer haul it’s really useful.
FG: What percent are 0-RTT in TLS.
MT: Like 1% when we have a ticket and we happen to need another connection.
BT: Can you say anything about how get tickets from?
MT: Nope we don’t know and don’t want to know.

