# Signature

## Theory

A signature is a hash that can be used to check the validity of data. The signature can be supplied separately from the data it validates, or in the case of CMS or SOAP it can be included in the same file. (where parts of that file contain the data and parts contain the signature).

The signature is used when integrity is important. It is intended to be an assurance that the data sent from Party-A to Party-B has not been altered. Therefore, Party-A signs the data by hashing the data and encrypting that hash using an asymmetric private key. Party-B can then verify the data by calculating the hash of the data and decrypting the signature to compare whether the two hashes are the same.

### RAW signatures

A raw signature is usually calculated by Party-A as follows:

* create a hash of the data (e.g., an SHA-256 hash)
* encrypt the hash using an asymmetric private key (e.g., a 2048-bit RSA key)
* (optionally) encode the encrypted binary hash using base64 encryption.

Party-B will also have to obtain the certificate with the public key. It is possible that this has been exchanged before. So there are at least 3 files involved: the data, the signature and the certificate.

### CMS signatures

A CMS signature is a standardized way of sending data + signature + certificate with the public key, all in one file from Party-A to Party-B. As long as the certificate is valid and not revoked, Party-B can use the supplied public key to verify the signature.

### SOAP signatures

A SOAP signature also contains data and the signature and, optionally, the certificate. All in an XML payload. The calculation of the hash of the data involves some special steps. This has to do with the fact that SOAP XML sent from system to system can introduce extra elements or timestamps. In addition, SOAP Signing offers the possibility to sign different parts of the message by different parties.

### E-mail signatures

Sending e-mails is not very difficult. You have to fill in some data and send them to a server that forwards them, and they will eventually reach their destination. However, it is possible to send emails with a FROM field that is not your own email address. To assure your recipient that you really sent that email, you can sign your email. A trusted third party will verify your identity and issue an email signing certificate. You install the private key in your email application and configure it to sign the emails you send. The certificate is issued to a specific email address and everyone else who receives this email will see an indication that the sender is verified, because their tools will verify the signature using the public certificate that was issued by the trusted third party.

### PDF or Word or other signatures

Adobe PDF documents and Microsoft Word documents are also examples of things that support signing. The signature is also within the same document as the data, so there is a description of what is part of the data and what is part of the metadata. Governments often send official documents with a PDF containing a certificate.
