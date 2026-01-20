 
APRS Messages
=============

The Polaric-aprsd implement authenticated and encrypted APRS messaging. It supports AES-256 based encryption with authentication tags, or authentication without encryption using message authentication codes (currently based on MD5). 

Messages have a *message-id*. Message-ids shouldn't be reused often. All messages are expected to be answered with an ACK or a REJ message. 

Authenticated APRS messages
---------------------------

The format of a message is as follows::

 :<recipient><content>#<MAC>{<messsage-id>

**recipient** 
    (as defined in the APRS standard), callsign of the recipient padded with spaces so that its length is excactly 10 characters.

**content**
    The message to be sent

**MAC**
    A checksum is computed by concatenating *a secret key*, the *sender callsign*, the *recipient callsign*, the *content* and the *message-id* and generating a MD5 hash from the resulting byte-string (the characters of the message are UTF-8 encoded before generating the hash). The resulting MD5 hash is *base-64* encoded and the first 8 bytes of the encoded hash is used as the message authentication code (MAC) sent with the message. Both the sender and receiver of a message compute the MAC. On receipt of a message a node should compute the MAC and compare it with the MAC received with the message. If not match, the message should be rejected.

**messsage-id** 
    (as defined in the APRS standard). It should be unique for each message. Polaric APRSD use a sequence-number that is incremented with each message.



Encrypted APRS messages
-----------------------

From v. 4.1 of *Polaric-aprsd* we may encrypt APRS messages to meet security requirements (confidentiality, integrity). The encryption scheme used also perform authentication of messages. It is possible to specify in the setup for which other servers this form of encryption is to be used. If on, the content of the message is encrypted using AES/GCM-SIV without padding. The AES-key is generated from the secret key (may be the same as used for authenticating messages). We use BKDF2 with HmacSHA256. A *salt* is used and should be different for each service that use the secret key as a crypto-key where one is specific for APRS-message encryption. When encrypting messages, an initialisation vector (IV) is used starting with the *message-id* and padding the rest of the IV with null so that its length is always 12 bytes. The encrypted message is encoded using base64.

For a reference implementation, see source code utils/AesGcmSivEncryption.java.

The format of a message is as follows::

 :<recipient><content>{<messsage-id>

**recipient** 
    (as defined in the APRS standard), callsign of the recipient padded with spaces so that its length is excactly 10 characters.

**content**
    Base64 representation of the ciphertext. Padding removed. 

**messsage-id** 
    (as defined in the APRS standard). It should be unique for each message. Polaric APRSD use a sequence-number that is incremented with each message.


.. note::
 If sensitive content, messages to be sent worldwide over APRS-IS *should* be encrypted! Before configuring your software-installation to encrypt messages to be sent over amateur radio, be sure to check the HAM radio regulations in the country in question. 

Splitting long messages
-----------------------

Encrypted messages are longer than plain-text messages because of the authentication-tag and base64-encoding. We therefore allow them to be split up and sent as parts. We put a semicolon as the last character a part-message to indicate that the next part should be expected in the following message (with the next message-id). If the *first* character is a semicolon this indicates that this follows the previous message. Semicolon is not used in base64-encoding. In the current version, we split messages into at most two parts. 

The second part is acked right away when received. The first part is acked when reconstructed, decrypted and accepted by the application (or rejected if decryption/authentication fails or the message for some other reason is not accepted). For example, a ciphertext "12345678" can be sent as two messages: "1234;}001" followed by ";5678}002". 

Acknowledgment messages
-----------------------

On receipt of a message a node should respond with an ACK or REJ message as described in the APRS protocol to indicate if the message was successfully delivered to the application and that a command was successfully executed or not. The sender should retry a message if an acknowledgement is not received within a certain time (but there should be a limit on the number of retries, e.g. 3). The recipient node should use the *message-id* to ensure that the same message is not delivered to the application more than once.

Destination address
-------------------

Aprs packets have a destination field. When APRS packets aren't adresses to specific stations, this field is often used to indicate which device or software is used. Polaric Server use ``APPSxx`` where xx indicate the version. This field can also be used to indicate that encryption or authentication is used and which version. ``APPSEx`` is used to indicate that encryption is used (the protocol described here). x can indicate which version starting with 1 and incremented when protocol changes significantly. ``APPSAx`` could be used in a similar way to indicate that authentication is used.   



Encrypted APRS position reports 
===============================

Encrypting APRS packets other than messages is a somewhat different story. There is no packet-type for that in the original APRS standard, so we use a user-defined (experimental) format (prefixed with curly brackets '}'). We use AES/GCM-SIV the same way as with APRS-messages. The AES-key is generated from a secret key or passphrase (may be the same as used for authenticating messages and/or APRS-messages). *Polaric Server* uses a salt for APRS-messages and another salt for APRS position reports. When encrypting messages, an initialisation vector (IV) is used starting with the *message-id* and padding the rest of the IV with null so that its length is always 12 bytes. The encrypted message is encoded using base91.

The format of an encrypted packet content is:: 

 }}:<content>

**content**
    Base91 representation of the ciphertext. 

A decrypted version of the packet can be constructed using the *from-callsign*, the *to-callsign* and the *path-information* from the received packet and by replacing the content-part (including '}}:' prefix) with the decrypted content. It is recommended to use the ``APPSEx`` to-address. If a '}}' prefix is used, it is *strongly* recommended. We will try to reserve a prefix for *Polaric Server* or the *Norwegian Radio Relay league*. 

*Polaric Server* (from v. 4.1) allows decrypting and encrypting position-reports or object-reports using this method. In principle all APRS messages can be encrypted this way. 

.. note::
 Before configuring your software-installation to encrypt messages to be sent over amateur radio, be sure to check the HAM radio regulations in the country in question. Please be sure that encryption is used only where legal.

