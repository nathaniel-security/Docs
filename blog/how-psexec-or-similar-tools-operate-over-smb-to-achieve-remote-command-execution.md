# How PsExec or similar tools operate over SMB to achieve remote command execution

* SMB Connection:
  * The tool first establishes an SMB connection to the target machine using valid credentials.
* Service Creation:
  * It creates a temporary service on the remote system. This is done by writing a small service executable to the ADMIN$ share (which maps to the Windows directory) on the target.
* Service Execution:
  * The tool then uses the Service Control Manager (SCM) to start this newly created service.
* Command Execution:
  * The service, when started, executes a command shell (cmd.exe) or other specified command.
* I/O Redirection:
  * The tool sets up named pipes over SMB to redirect the input and output of this command shell back to the attacker's machine.
* Cleanup:
  * After the session is established, the temporary service is typically deleted to remove traces of the intrusion.
* The key points here are:
  * SMB is used for file transfer (uploading the service executable) and for creating named pipes for I/O redirection.
  * The Windows Service Control Manager is leveraged to execute code with SYSTEM privileges.
  * Named pipes provide a way to tunnel command execution and results through the SMB protocol.
