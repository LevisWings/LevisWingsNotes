# Privilege escalation paths

## Types of privileges

* Getting SYSTEM
* Becoming another user
* Changing IL (Integrity level)
* Abusing tokens.
* Gaining more privileges

## Paths

|              Before             |                      After                      |
| :-----------------------------: | :---------------------------------------------: |
| Non-Admin Medium IL No Password | <p>Non-Admin</p><p>Medium IL</p><p>Password</p> |
|   Admin Medium IL No Password   |   <p>Admin</p><p>Medium IL</p><p>Password</p>   |
|       Non-Admin Medium IL       |           <p>Admin</p><p>Medium IL</p>          |
|   <p>Admin</p><p>Medium IL</p>  |            <p>Admin</p><p>High IL</p>           |

### Paths to SYSTEM

* Admin High IL -> SYSTEM
* Non-Admin and Medium IL -> SYSTEM
* Service Account -> SYSTEM
