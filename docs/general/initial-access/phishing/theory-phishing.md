# Theory (Phishing)

## Email delivery

There are 3 specific protocols involved to facilitate the outgoing and incoming email messages, and they are briefly listed below.

* SMTP (Simple Mail Transfer Protocol) - It is utilized to handle the sending of emails.
* POP3 (Post Office Protocol) - Is responsible transferring email between a client and a mail server.
* IMAP (Internet Message Access Protocol) - Is responsible transferring email between a client and a mail server.

You should have noticed that both POP3 and IMAP have the same definition. But there are differences between the two:

#### POP3

* Emails are downloaded and stored on a single device.
* Sent messages are stored on the single device from which the email was sent.
* Emails can only be accessed from the single device the emails were downloaded to.
* If you want to keep messages on the server, make sure the setting "Keep email on server" is enabled, or all messages are deleted from the server once downloaded to the single device's app or software.

#### IMAP

* Emails are stored on the server and can be downloaded to multiple devices.
* Sent messages are stored on the server.
* Messages can be synced and accessed across multiple devices.

Example of an email delivery:

![Image from TryHackMe: https://tryhackme.com/room/phishingemails1tryoe](../../../.gitbook/assets/email\_delivery\_example.png)

1. Alexa composes an email to Billy (`billy@johndoe.com`) in her favorite email client. After she's done, she hits the send button.
2. The **SMTP** server needs to determine where to send Alexa's email. It queries **DNS** for information associated with `johndoe.com`.&#x20;
3. The **DNS** server obtains the information `johndoe.com` and sends that information to the **SMTP** server.&#x20;
4. The **SMTP** server sends Alexa's email across the Internet to Billy's mailbox at `johndoe.com`.
5. In this stage, Alexa's email passes through various **SMTP** servers and is finally relayed to the destination **SMTP** server.&#x20;
6. Alexa's email finally reached the destination **SMTP** server.
7. Alexa's email is forwarded and is now sitting in the local **POP3/IMAP** server waiting for Billy.&#x20;
8. Billy logs into his email client, which queries the local **POP3/IMAP** server for new emails in his mailbox.
9. Alexa's email is copied (**IMAP**) or downloaded (**POP3**) to Billy's email client.&#x20;

## Email headers/body

{% embed url="https://mediatemple.net/community/products/all/204643950/understanding-an-email-header" %}

### Analyze headers

{% embed url="https://mha.azurewebsites.net" %}

