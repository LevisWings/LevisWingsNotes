# To-do

### To-do

* [ ] Add Python scripts to communicate with other services/protocols (SMB, SMTP, etc.).
* [ ] Add advice that when we get an interactive console on the machine, look for credentials of the web application (WordPress, gitea, etc.) as there are cases where a user has a private project or notes in that application.
* [ ] Add advice to look for vulnerabilities in services and software (even in the strangest ones like SSH).
* [ ] Add advice to rebuild a whole web page (if we can do it) in a directory for convenience.
* [ ] Privilege Escalation: If there is a Python or PHP script that is being executed by root every so often, but we do not have write access, do not abandon a possible escalation, because if that script includes or imports another file (import Play from settings (Python) or include('/var/www/html/test.php') from PHP), it is possible to escalate privileges.
* [ ] Python crawler: A script that from a Python file, search by "import" or "from" statements and with that information, search for other possible .py files.
*   [ ] SQL injection in cookies (either a normal one found in the Cookie header or in the JWT).

    Translated with www.DeepL.com/Translator (free version)
