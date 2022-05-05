# Python Virtual Environment

A Python virtual environment is a self-contained directory tree that includes a Python installation and the set of additional packages that are installed.

The virtual environment is one of the most essential tools for a Python developer. Using one, you can separate projects that have different needs. For example, you can use one virtual environment for package inspection projects and a different one for binary analysis projects. By having separate environments, you keep your projects simple and clean. This ensures that each environment can have its own set of dependencies and modules without disrupting any of your other projects.

### Practice

```bash
mkdir test
python3 -m venv <ENV NAME>
source <ENV NAME>/bin/activate # To active the virtual environment

pip3 install <PACKAGE>

deactivate # To exit the virtual environment
```
