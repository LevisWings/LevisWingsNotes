# Web Pentest Methodology

* [ ] Start by **identifying** the **technologies** used by the web server. Look for **tricks** to keep in mind during the rest of the test if you can successfully identify the tech.
  * [ ] Any **known vulnerability** of the version of the technology?
  * [ ] Using any **well known tech**? Any **useful trick** to extract more information?
  * [ ] Any **specialised scanner** to run (like wpscan)?
* [ ] **Subdomains enumeration**: Each subdomain represents a new angle to attack the network.
  * [ ] We can also perform **fuzzing on vhosts**.
* [ ] Launch **general purposes scanners**. You never know if they are going to find something or if the are going to find some interesting information.
* [ ] Start with the **initial checks**: **robots**, **sitemap**, **404** error and **SSL/TLS scan** (if HTTPS).
* [ ] Start **spidering** the web page: It's time to **find** all the possible **files, folders** and **parameters being used.** Also, check for **special findings**.
  * [ ] _Note that anytime a new directory is discovered during brute-forcing or spidering, it should be spidered._
* [ ] **Directory Brute-Forcing**: Try to brute force all the discovered folders searching for new **files** and **directories**.
  * [ ] _Note that anytime a new directory is discovered during brute-forcing or spidering, it should be Brute-Forced._
  * [ ] If there is a similar file name format (e.g., all file names start with a capital letter), then you should take this into account when fuzzing. More info: **File or directory name format**.
* [ ] **Backups checking**: Test if you can find **backups** of **discovered files** appending common backup extensions.&#x20;
* [ ] **Brute-Force parameters**: Try to **find hidden parameters**.
* [ ] Once you have **identified** all the possible **endpoints** accepting **user input**, check for all kind of **vulnerabilities** related to it.
