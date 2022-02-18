## FTP checklist
* [ ] Identify the FTP version. Is it vulnerable?

If you have credentials, authenticate. If you do not have them:
  * [ ] Do you allow authentication with the user anonymous?
  * [ ] Do you have a user? Try brute force. This method should be used as a last resort if you can't find anything in other services.
If we have an FTP session:
  * [ ] Can't list files? Remember to use passive mode.
  * [ ] What directory are we in?
  * [ ] Can we download files?
    * [ ] If there are any interesting files, download them.
  * [ ] Check the permissions of all files and directories.
  * [ ] Can we upload or overwrite files?
    * [ ] Do you suspect that a user or task is interacting with the FTP service? If so, we can overwrite a script or upload an exploit that abuses a task running on that service.
    * [ ] Are there configuration files? Download them and see if they may be synchronized with other services (for example, if there is an http configuration file that denies access to a user, we can overwrite the file to allow access).
    * [ ] Is the FTP server synchronized to another service (such as HTTP)? Try uploading a webshell with the programming language being used in the backend.
