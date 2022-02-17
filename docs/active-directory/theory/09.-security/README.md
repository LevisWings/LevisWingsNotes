# 📓 09. Security

Now that we have a more clear idea of the Active Directory elements, let's talk about the topics that are more related with security in Active Directory. Security is based on the following pillars:

### Address resolution

The ability for users and machines to resolve addresses of another computers in order to establish connections with them. If an attacker can control the address resolutions, then she could perform Person in The Middle attacks or make that users send their credentials to attacker controlled machines.

### Authentication

The ability to identify the user that is accessing to a computer of service. If an attacker can get the user credentials or authentication is bypassed, then she will be able to identify herself as the user and perform actions on its behalf.

### Authorization

The ability to identify what actions can be performed by an user. If the user permissions are incorrectly configured, she may be able to perform privileged actions.

Let's discuss about these fundamentals and how they are implemented in Active Directory.
