# Download a git from the tree

Assume the following part of a repository:

```bash
https://github.com/internetwache/GitTools/tree/master/Extractor
```

If we try to do a `git clone` it will not work since we are in a tree and branch (in this case, master). To be able to download from there, we have to replace the tree and branch (**tree/master**) with a **trunk** and do a `svn checkout` at the beginning:

```bash
svn checkout "https://github.com/internetwache/GitTools/trunk/Extractor"
```
