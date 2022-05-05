# SSL Pinning

SSL Pinning is a security methodology used to ensure that application traffic is not being intercepted (Man-In-The-Middle).

* Some mobile applications VERIFY that the traffic received comes from a KNOWN certificate.
* We can import a certificate into the phone as a root or user certificate, but it still might not be trusted by the application.
* This can cause our application to crash when we try to intercept network traffic.

![](../../../../.gitbook/assets/ssl\_pinning.png)

From a pentester's perspective, this can make our job a bit more difficult. We want to see the live application traffic, see the parameters passed and edit them if possible!

### Android interception process

1. Start the proxy software
2. Configure the proxy software
3. Configure the Emulator Proxy (or wifi configuration for a physical device).
4. Intercept HTTP traffic.
5. Import CA certificate.
6. Trust CA certificate in the Android certificate store.
7. Intercept HTTPS traffic = Win?! (or be embarrassed by SSL Pinning)
8. If embarrassed by SSL Pinning, try Objection/Frida
   1. [Patching Applications Automatically using Objection](patching-applications-automatically-using-objection.md)
   2. [Patching Applications Manually](patching-applications-manually.md)
