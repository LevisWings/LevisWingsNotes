# tshark

```bash
tshark -r <.cap>
tshark -r <.cap> | wc -l # Total number of packages in the capture.
tshark -r <.cap> -Y "<DISPLAY FILTER>"
tshark -r <.cap> -Y "dns.qry.type == 1" # DNS
tshark -r <.cap> -Y "dns.qry.type == 1" -T fields -e dns.qry.name # Extract domain names
tshark -r <.cap> -Y "(dns.flags.response == 0) and (ip.src == <IP>)" | wc -l # All DNS queries
```

### Tricks

An easy way to identify **field names** (`-T fields -e <FIELD NAME>`) in Wireshark is to navigate to the Packet Details in the capture, highlight the interesting field, then view the bottom left corner.

!["Query Name (dns.qry.name)"](../.gitbook/assets/field\_name\_wireshark.png)
