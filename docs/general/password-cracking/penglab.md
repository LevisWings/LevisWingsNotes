# penglab

**Penglab** is a ready-to-install setup on Google Colab for cracking hashes with an incredible power, really useful for CTFs.

### How to use it ?

1. Go on : [https://colab.research.google.com/github/mxrch/penglab/blob/master/penglab.ipynb](https://colab.research.google.com/github/mxrch/penglab/blob/master/penglab.ipynb)
2. Select "**Runtime**", "**Change runtime type**", and set "**Hardware accelerator**" to **GPU**.
3. Change the config by setting "**True**" at tools you want to install. Example with hashcat and rockyou:

```python
#Penglab (Abuse of Google Colab for fun and profit)
#by mxrch

#Choose what you want to install
hashcat = True # Changed to True
john = False
hydra = False

ssh = False 
python_shell = False
bash_shell = False

wordlists_dir = "wordlists"

rockyou = True # Changed to True
hashesorg2019 = False

#---------------------------

if (python_shell and bash_shell) or (bash_shell and ssh) or (ssh and python_shell) :
    print("Please do a choice")
    exit()
```

4\. Add the following lines of code in the wordlists part to load the hashes:

```python
mode = 3200

hashes = """
<HASH1>
<HASH2>
...
"""

with open("hashes", "w") as f:
  f.write(hashes)
```

![](../../.gitbook/assets/hash\_file\_penglab.png)

5\. At the bottom, in the "# Put your code here" section, enter the following command to execute hashcat:

```
!hashcat -m {mode} hashes /content/wordlists/rockyou.txt --user
```

4\. Select "**Runtime**" and "**Run all**" !

