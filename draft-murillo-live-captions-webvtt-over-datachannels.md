---
docname: draft-murillo-live-captions-webvtt-over-datachannels-00
title: Live Captions: WebVTT over datachannels
abbrev: webvtt-datachannels
category: std
ipr: trust200902

keyword: WebRTC live captions datachannels webvtt

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: S. Murillo
    name: Sergio Garcia Murillo
    organization: Millicast
    email: sergio.garcia.murillo@cosmosoftware.io


normative:
  RFC2119:
  RFC7675:
  RFC8840:
  RFC8853:
  RFC8863:

--- abstract

   This document specifies how a Web Real-Time Communication (WebRTC)
   data channel can be used as a transport mechanism for live captioning 
   using the WebVTT the Web Video Text Tracks Format and how the Session Description Protocol
   (SDP) offer/answer mechanism can be used to negotiate such a data
   channel.

--- middle

# Introduction

   The WebVTT: The Web Video Text Tracks Format specification [WebVTT] specifies how to mark up external text track resources in connection with the HTML <track> element. 
   WebVTT files provide captions or subtitles for video content, and also text video descriptions, chapters for content navigation, and more generally any form of metadata that is time-aligned with audio or video content.

   However, the WebVTT file format is not suited for live captioning, which requires captions to be generated and transferred in real time to the viewers.
   
   This document specifies how to incrementally encode and transfer WebVTT cues so they can be both rendered in real time by a WebVTT capable player and also stored on a compatible WebVTT file format for recording.

   In this document, a WebVTT data channel refers to a WebRTC data
   channel for which the instantiated subprotocol is "webvtt" and where
   the channel is negotiated using the SDP offer/answer mechanism
   [RFC8864].

      |  NOTE: The decision to transport real-time text using a WebRTC
      |  data channel instead of using RTP-based transport [RFC4103] is
      |  motivated by use case "U-C 5: Real-time text chat during an
      |  audio and/or video call with an individual or with multiple
      |  people in a conference"; see Section 3.2 of [RFC8831].

   Live caption text is intended to be entered by human users via a
   keyboard, handwriting recognition, voice recognition, or any other
   input method.

   Section 3 defines the generic data channel properties for a WebVTT
   data channel, and Section 4 defines how they are conveyed in an SDP
   'dcmap' attribute.  While this document defines how to negotiate a
   WebVTT data channel using the SDP offer/answer mechanism [RFC8864],
   the generic WebVTT and gateway considerations defined in Sections 3,
   5, and 6 of this document can also be applied when a WebVTT data
   channel is established using another mechanism (e.g., the mechanism
   defined in [RFC8832]).  Section 5 of [RFC8864] defines the mapping
   between the SDP 'dcmap' attribute parameters and the protocol
   parameters used in [RFC8832].


# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in {{!RFC2119}}.

# WebRTC Data Channel Considerations

   The following WebRTC data channel property values [RFC8831] apply to
   a WebVTT data channel:

              +--------------------------+--------------------+
              | Subprotocol Identifier   | webvtt             |
              +--------------------------+--------------------+
              | Transmission reliability | reliable           |
              +--------------------------+--------------------+
              | Transmission order       | in-order           |
              +--------------------------+--------------------+
              | PPIDs                    | WebRTC String      |
              +--------------------------+--------------------+
              | Label                    | See Section 4.1    |
              +--------------------------+--------------------+

#  WebVTT cues over datachannels

This document allows sending a single WebVTT queue over a datachannal message using UTF-8 encoding.

    00:00.000 --> 01:24.000
    Introduction

When using WebVTT from a file during the playback, the timestamps of the WebVTT queues are relative to the start of the playback media. However, when doing real time captioning, the playback time may not be the known when generating the captions and will be different if multiple people are watching the same live stream simultaneously.

In order to allow to reuse the same WebVTT cues for multiple players witout requiring rewriting on the server side, the format for the WebVTT cues is modified to use absolute timestamps in EPOCH time instead of relative ones.

    1649774427571 --> 1649774428771
    Introduction
    
How to synchronize the sender timestamps with the receiver timestamp if the media and captions traverses multiple hops is outside of the scope of this document.

It is possible to incrementally transmit captions before they are finished by sending multiple datachannel messages with WebVTT cueues with the same initial timestamp


    1649774427571 --> 1649774428771
    This is ...
    
    1649774427571 --> 1649774429771
    This is an incremental ...
    
    1649774427571 --> 1649774430771
    This is an incremental caption
    
The receiver will override the content of the previous received webvtt cue and store the last received one
   

#  SDP Considerations

   The generic SDP considerations, including the SDP offer/answer
   procedures [RFC3264], for negotiating a WebRTC data channel are
   defined in [RFC8864].  This section, and its subsections, define the
   SDP considerations that are specific to a WebVTT data channel,
   identified by the 'subprotocol' attribute parameter, with a "webvtt"
   parameter value, in the 'dcmap' attribute.

##  Use of the 'dcmap' Attribute

   An offerer and answerer MUST, in each offer and answer, include an
   SDP 'dcmap' attribute [RFC8864] in the SDP media description
   ("m=" section) [RFC4566] describing the Stream Control Transmission
   Protocol (SCTP) association [RFC4960] used to realize the WebVTT data
   channel.

   The offerer and answerer MUST include the 'subprotocol' attribute
   parameter, with a "webvtt" parameter value, in the 'dcmap' attribute.

   The offerer and answerer MAY include the 'priority' attribute
   parameter and the 'label' attribute parameter in the 'dcmap'
   attribute value, as specified in [RFC8864].

      |  NOTE: As specified in [RFC8831], when a data channel is
      |  negotiated using the mechanism defined in [RFC8832], the
      |  'label' attribute parameter value has to be the same in both
      |  directions.  That rule also applies to data channels negotiated
      |  using the mechanism defined in this document.

   The offerer and answerer MUST NOT include the 'max-retr' or 'max-
   time' attribute parameter in the 'dcmap' attribute.  If either of
   those attribute parameters is received in an offer, the answerer MUST
   reject the offer.  If either of those attribute parameters is
   received in an answer, the offerer MUST NOT accept the answer.
   Instead, the answerer MUST take appropriate actions, e.g., by sending
   a new offer without a WebVTT data channel or by terminating the
   session.

   If the 'ordered' attribute parameter is included in the 'dcmap'
   attribute, it MUST be assigned the value 'true'.

   Below is an example of the 'dcmap' attribute for a WebVTT data channel
   with stream id=3 and without any label:

      a=dcmap:3 subprotocol="webvtt"
      
##  Use of the 'dcsa' Attribute

   An offerer and answerer can, in each offer and answer, include one or
   more SDP 'dcsa' attributes [RFC8864] in the "m=" section describing
   the SCTP association used to realize the WebVTT data channel.

   If an offerer or answerer receives a 'dcsa' attribute that contains
   an SDP attribute whose usage has not been defined for a WebVTT data
   channel, the offerer or answerer should ignore the 'dcsa' attribute,
   following the rules in Section 6.7 of [RFC8864].
   
###  Live Caption Languages

   'dcsa' attributes can contain the SDP 'hlang-send' and 'hlang-recv'
   attributes [RFC8373] to negotiate the language to be used for the
   real-time text conversation.

   For a WebVTT data channel, the modality is "written" [RFC8373].

###  Live Caption Direction

   'dcsa' attributes can contain the SDP 'sendonly', 'recvonly',
   'sendrecv', and 'inactive' attributes [RFC4566] to negotiate the
   direction in which text can be transmitted .

      |  NOTE: A WebRTC data channel is always bidirectional.  The usage
      |  of the 'dcsa' attribute only affects the direction in which
      |  implementations are allowed to transmit captions on a WebVTT data
      |  channel.

   The offer/answer rules for the direction attributes are based on the
   rules for unicast streams defined in [RFC3264], as described below.
   Note that the rules only apply to the direction attributes.

   Session-level direction attributes [RFC4566] have no impact on a
   WebVTT data channel.
   
##  Generating an Offer

   If the offerer wishes to both send and receive text on a WebVTT data
   channel, it SHOULD mark the data channel as sendrecv with a
   'sendrecv' attribute inside a 'dcsa' attribute.  If the offerer does
   not explicitly mark the data channel, an implicit 'sendrecv'
   attribute inside a 'dcsa' attribute is applied by default.

   If the offerer wishes to only send text on a WebVTT data channel, it
   MUST mark the data channel as sendonly with a 'sendonly' attribute
   inside a 'dcsa' attribute.

   If the offerer wishes to only receive text on a WebVTT data channel,
   it MUST mark the data channel as recvonly with a 'recvonly' attribute
   inside a 'dcsa' attribute.

   If the offerer wishes to neither send nor receive text on a WebVTT
   data channel, it MUST mark the data channel as inactive with an
   'inactive' attribute inside a 'dcsa' attribute.

   If the offerer has marked a data channel as sendrecv (or if the
   offerer did not explicitly mark the data channel) or recvonly, it
   MUST be prepared to receive WebVTT data as soon as the state of the
   WebVTT data channel allows it.

##  Generating an Answer

   When the answerer accepts an offer and marks the direction of the
   text in the corresponding answer, the direction is based on the
   marking (or the lack of explicit marking) in the offer.

   If the offerer either explicitly marked the data channel as sendrecv
   or did not mark the data channel, the answerer SHOULD mark the data
   channel as sendrecv, sendonly, recvonly, or inactive with a
   'sendrecv', 'sendonly', 'recvonly', or 'inactive' attribute,
   respectively, inside a 'dcsa' attribute.  If the answerer does not
   explicitly mark the data channel, an implicit 'sendrecv' attribute
   inside a 'dcsa' attribute is applied by default.

   If the offerer marked the data channel as sendonly, the answerer MUST
   mark the data channel as recvonly or inactive with a 'recvonly' or
   'inactive' attribute, respectively, inside a 'dcsa' attribute.

   If the offerer marked the data channel as recvonly, the answerer MUST
   mark the data channel as sendonly or inactive with a 'sendonly' or
   'inactive' attribute, respectively, inside a 'dcsa' attribute.

   If the offerer marked the data channel as inactive, the answerer MUST
   mark the data channel as inactive with an 'inactive' attribute inside
   a 'dcsa' attribute.

   If the answerer has marked a data channel as sendrecv or recvonly, it
   MUST be prepared to receive data as soon as the state of the WebVTT
   data channel allows transmission of data.

##  Offerer Receiving an Answer

   When the offerer receives an answer to the offer and the answerer has
   marked a data channel as sendrecv (or the answerer did not mark the
   data channel) or recvonly in the answer, the offerer can start
   sending WebVTT data as soon as the state of the WebVTT data channel
   allows it.  If the answerer has marked the data channel as inactive
   or sendonly, the offerer MUST NOT send any WebVTT data.

   If the answerer has not marked the direction of a WebVTT data channel
   in accordance with the procedures above, it is RECOMMENDED that the
   offerer not process that scenario as an error situation but rather
   assume that the answerer might both send and receive WebVTT data on
   the data channel.

##  Modifying the Text Direction

   If an endpoint wishes to modify a previously negotiated text
   direction in an ongoing session, it MUST initiate an offer that
   indicates the new direction, following the rules in Section 4.2.3.1.
   If the answerer accepts the offer, it follows the procedures in
   Section 4.2.3.2.
   
##  Modifying the Caption Languages

   If an endpoint wishes to modify a previously negotiated captioning language in an ongoing session, it MUST initiate an offer that
   indicates the new language dcsa attribute, following the rules in Section 4.2.3.1.
   If the answerer accepts the offer, it follows the procedures in
   Section 4.2.3.2.

##  Examples

   Below is an example of an "m=" section of an offer for two WebVTT data
   channel offering live captions in Spanish and
   English, and an "m=" section in the associated answer accepting
   English.

   Offer:

      m=application 911 UDP/DTLS/SCTP webrtc-datachannel
      c=IN IP6 2001:db8::3
      a=max-message-size:1000
      a=sctp-port 5000
      a=setup:actpass
      a=dcmap:2 label="Closed Captions";subprotocol="webvtt"
      a=dcsa:2 hlang-send:en es
      a=dcsa:2 sendonly

   Answer:

      m=application 2004 UDP/DTLS/SCTP webrtc-datachannel
      c=IN IP6 2001:db8::1
      a=max-message-size:1000
      a=sctp-port 6000
      a=setup:passive
      a=dcmap:2 subprotocol="webvtt"
      a=dcsa:2 recvonly
      a=dcsa:2 hlang-recv:en


   Below is an example of an "m=" section of an offer for two WebVTT data
   channels, one sending english captions and another with spanish ones.

   Offer:

      m=application 1400 UDP/DTLS/SCTP webrtc-datachannel
      c=IN IP6 2001:db8::3
      a=max-message-size:1000
      a=sctp-port 5000
      a=setup:actpass
      a=dcmap:2 label="English Closed Captions";subprotocol="webvtt"
      a=dcsa:2 hlang-send:en
      a=dcsa:2 sendonly
      a=dcmap:3 label="Spanish Closed Captions";subprotocol="webvtt"
      a=dcsa:3 hlang-send:es
      a=dcsa:3 sendonly

   Answer:

      m=application 2400 UDP/DTLS/SCTP webrtc-datachannel
      c=IN IP6 2001:db8::1
      a=max-message-size:1000
      a=sctp-port 6000
      a=setup:passive
      a=dcmap:2 subprotocol="webvtt"
      a=dcsa:2 recvonly
      a=dcmap:3 subprotocol="webvtt"
      a=dcsa:3 recvonly
      
#  Security Considerations

   The generic WebRTC security considerations are defined in [RFC8826]
   and [RFC8827].

   The generic security considerations for WebRTC data channels are
   defined in [RFC8831].  As data channels are always encrypted by
   design, the WebVTT ata channels will also be encrypted.

   The generic security considerations for negotiating data channels
   using the SDP offer/answer mechanism are defined in [RFC8864].  There
   are no additional security considerations specific to WebVTT data
   channels.

#  IANA Considerations

##  Subprotocol Identifier "webvtt"

   Per this document, the subprotocol identifier "webvtt" has been added
   to the "WebSocket Subprotocol Name Registry" as follows:

   Subprotocol Identifier:  webvtt

   Subprotocol Common Name:  webvtt over datachannels

   Subprotocol Definition:  TBD

   Reference:  RFC TBD

--- back


