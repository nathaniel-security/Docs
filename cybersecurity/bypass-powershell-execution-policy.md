# Bypass Powershell Execution Policy

* It is NOT a security measure,
  * it is present to prevent user from accidently executing scripts.

### Ways to Bypass

```
powershell -Executionpolicy bypass
```

```
powershell -c <cmd>
```

```
powershell -encodedcommand  $env : PSExecutionPolicyPreference="bypass"
```
