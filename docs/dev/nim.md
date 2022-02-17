# NIM

## Compilation

```shell
# Console/GUI app:
nim c -d=mingw --app=console --cpu=amd64 launcher.nim
nim c -d=mingw --app=gui --cpu=amd64 launcher.nim
# For AV evasion:
nim c -d:mingw -d:release --app=gui --deadCodeElim:on --opt:size --stackTrace:off --cpu=amd64 launcher.nim
```

### AV Evasion



```nim
#[
    Author: Marcello Salvati, Twitter: @byt3bl33d3r
    Modified: leviswings (alias), Twitter: @Levis_Wings
    License: BSD 3-Clause
    References:
        - https://gist.github.com/cpoDesign/66187c14092ceb559250183abbf9e774
]#

import winim/clr

var Automation = load("System.Management.Automation")
var RunspaceFactory = Automation.GetType("System.Management.Automation.Runspaces.RunspaceFactory")

var runspace = @RunspaceFactory.CreateRunspace()

runspace.Open()

var pipeline = runspace.CreatePipeline()
pipeline.Commands.AddScript("Get-Process")

pipeline.Invoke()
runspace.Close()
```
