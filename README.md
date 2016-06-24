# runAs

## Ccommand line arguments

| Argument | Description | Optional | Default value|
| ------------- |:-------------:|:-------------:|:-------------:|
| -u:user_name  | *"user"* or *"domain\user"* or *"user@domain"* |||
| -p:password   | user's password | X | empty |
| -w:working_directory | current directory | X | empty |
| -b:exit_code_base | base number for exit code | X | -100000 |
| -e:environment_var | set an environment variable in the format "name=value" | X | |
| -l:log_level | logging level (*debug*\|*normal*\|*errors*\|*off*) | X | *normal* |
| -il:integrity_level | [integrity level](https://msdn.microsoft.com/en-us/library/bb625963.aspx)(*auto*\|*untrusted*\|*low*\|*medium*\|*medium_plus*\|*high*) | X | *auto* |
| -i:inheritance_mode |- set to **off** when environment variables should not be inherited from the parent process<br/>- set **on** when the environment variables should be inherited from the  parent process<br/>- set to **auto** when some environment variables should be inherited from the parent process| X | *auto* |
| -c:configuration_file | text file, containing any configuration arguments | | |
| executable | executable file | | |
| command_line_args | command line arguments, the maximum total length of *executable* and *command_line_args* is 1024 characters | X | empty |

## Some comments

Each account has a set of privileges is like a set of abilities.
Windows integrity level is like a filter of abilities.

So your account could have administrative privileges, but _low_ integrity level. In this case the most of privileges will be filtered. For example: administrator in the Chrome browser.

Another your account could have standard user privileges, but _high_ integrity level. In this case the most of privileges will not be filtered. For example: some Windows service working under the standard user account.

Windows API provides 2 ways to do _"runAs"_:
_CreateProcessAsUser_ (1) and _CreateProcessWithLogonW_ (2)

* (1) does it directly. The integrity level can be **_"High"_** (if primary security access token is used).
* (2) does it via dedicated logon service. The integrity level can be less or equal **_"Medium"_** in common cases. But this service can make it higher via the "Admin elevation" dialog, but it is not our case.

To use (1) caller **should have _SE_ASSIGNPRIMARYTOKEN_NAME_** privilege to replace a filtered (by Windows core) security access token by a primary (not filtered) security access token with the full set of privileges ("High" integrity leve). See this [page](https://msdn.microsoft.com/ru-ru/library/windows/desktop/ms682429(v=vs.85).aspx).

To use (2) caller **should have a logon SID**. See [this](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682431(v=vs.85).aspx).
You cannot call _CreateProcessWithLogonW_ from a process that is running under the _"LocalSystem"_ account, because the function uses the logon SID in the caller token, and the token for the _"LocalSystem"_ account does not contain this SID. As an alternative, use the (1) or use other accounts.
