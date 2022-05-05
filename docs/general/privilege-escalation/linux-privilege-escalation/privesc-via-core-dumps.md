# PrivEsc via Core Dumps

## Theory

A core dump, memory dump, crash dump, system dump, or ABEND dump consists of the recorded state of the working memory of a computer program at a specific time, generally when the program has crashed or otherwise terminated abnormally.

If we have a binary that has privileges to read any file and stores its contents in a buffer (memory), then we can "lock" the program, display the coredump to observe its contents.

### Ubuntu core dump (apport)

If any process in the system dies due to a signal that is commonly referred to as a 'crash' (segmentation violation, bus error, floating point exception, etc.), or e. g. a packaged Python application raises an uncaught exception, the **apport** backend is automatically invoked. It produces an initial crash report in a file in `/var/crash/` (the file name is composed from the name of the crashed executable and the user id).

### prctl (PR\_SET\_DUMPABLE)

**PR\_SET\_DUMPABLE**: Set the state of the "dumpable" attribute, which determines whether core dumps are produced for the calling process upon delivery of a signal whose default behavior is to produce a core dump.

## Practice

### Ubuntu (apport)

{% code title="Console #1" %}
```bash
# Step 1:
./binary-exaple
# Step 4:
apport-unpack /var/crash/<COREDUMP> <path/to/save>
# Step 5:
strings <FILE>
```
{% endcode %}

{% code title="Console #2" %}
```bash
# Step 2:
ps -aux | grep "binary-example"
# Step 3:
kill -BUS <PID>
```
{% endcode %}
