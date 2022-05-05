# Set Proxy

## Example with Burpsuite

Configure a new listener in Burpsuite:

{% hint style="danger" %}
The "**Bind to address**" option can be all interfaces but if you are using a VPN, use that IP.
{% endhint %}

![Port 8082 - All interfaces](../../../.gitbook/assets/burpsuite\_set\_listener\_android.png)

We set the Proxy in Android Studio:

{% tabs %}
{% tab title="adb (recommended)" %}
```bash
adb shell settings put global http_proxy :0 # Set "No proxy"
adb shell settings put global http_proxy <VPN_IP>:8082 # Set proxy
```
{% endtab %}

{% tab title="GUI (android-studio)" %}
![](../../../.gitbook/assets/burpsuite\_set\_listener\_android.png)
{% endtab %}
{% endtabs %}

We generate a .CER certificate from BurpSuite:

![](../../../.gitbook/assets/burpsuite\_generate\_cer.png)

![](../../../.gitbook/assets/burpsuite\_generate\_cer2.png)

We drag the .cer certificate to the Android phone and then go to import it:

{% tabs %}
{% tab title="android-studio" %}
![](../../../.gitbook/assets/install\_burpsuite\_cer\_android1.png)

![](../../../.gitbook/assets/install\_burpsuite\_cer\_android2.png)

![](../../../.gitbook/assets/install\_burpsuite\_cer\_android3.png)

![](../../../.gitbook/assets/install\_burpsuite\_cer\_android4.png)

![](../../../.gitbook/assets/install\_burpsuite\_cer\_android5.png)
{% endtab %}
{% endtabs %}
