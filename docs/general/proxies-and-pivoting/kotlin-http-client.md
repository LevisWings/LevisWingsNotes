# kotlin-http-client

HTTP client with ktor, created by "Hydragyrum".

### [Download](../../misc/tools.md#kotlin-http-client)

### Use

```bash
# GET requets:
java -jar http-client.jar -X GET <URL>
# Download file:
java -jar http-client.jar http://<LHOST>/port-scanner.jar -o port-scanner.jar
# Verbose:
java -jar http-client.jar -v <URL>
# OPTIONS method:
java -jar http-client.jar -v -X OPTIONS <URL>
# Header:
java -jarhttp-client.jar -v -H '<HEADER>' <URL>
```
