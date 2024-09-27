# Difference between actively logged on users, locally logged on users and remotely logged users

### **Actively Logged On Users**

These are users who have an active session on the system, meaning they are currently logged in and interacting with the machine. These users may be connected either locally or remotely, but the key factor is that they are actively using the system.

* **Context:** Could be either local or remote, but the session is currently live and in use.
* **Example:** A user is actively using their workstation or server to perform tasks like running applications or accessing resources.
* **Methods:** This could be via the local console, Remote Desktop, or any other session where the user is interacting in real time.

### **Locally Logged On Users**

These are users who have logged onto the machine physically (or locally) by sitting at the machine or workstation.

* **Context:** The user is logged in directly at the system (i.e., physically present at the machine or using local resources).
* **Example:** A user logs in to their desktop computer at their desk using their username and password.
* **Common Use Case:** This applies to users in an office setting using desktop computers or laptops without any remote connections involved.

**Key Characteristics:**

* **Physical access:** The user is physically present at the machine.
* **Method:** The user logged in via the machine’s local interface (e.g., the keyboard and monitor of the computer).

### **Remotely Logged On Users**

These are users who have logged onto the system from a different physical location using remote access technology, such as Remote Desktop Protocol (RDP), Virtual Private Network (VPN), or other remote access tools.

* **Context:** The user accesses the system over the network (LAN, WAN, Internet) rather than being physically present at the machine.
* **Example:** A user logs in to a server using Remote Desktop from another computer at home or another office.
* **Common Use Case:** Remote workers, system administrators, or technicians who need access to systems without being physically present.

**Key Characteristics:**

* **Remote access:** The user is not physically present but is connected via a network.
* **Method:** The user logged in through tools like RDP, VPN, or SSH (for Linux/Unix systems).

### Key Differences

| **Factor**        | **Actively Logged On Users**                                                    | **Locally Logged On Users**                                                     | **Remotely Logged On Users**                                                                    |
| ----------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Definition**    | Users who are actively interacting with the system, either locally or remotely. | Users physically present and logged on to the system at its physical interface. | Users connected remotely over a network.                                                        |
| **Access Method** | Can be either local or remote.                                                  | Direct physical access to the machine.                                          | Accessed via network using remote access tools like RDP or VPN.                                 |
| **Use Case**      | Anyone currently interacting with the system.                                   | Users at their workstations.                                                    | Users accessing servers or workstations remotely (e.g., work from home, server administration). |
| **Example**       | A user working on their workstation or remotely managing a server.              | A user typing on their desktop computer in the office.                          | A user connecting to a server via Remote Desktop from home.                                     |

### Summary:

* **Actively logged on users** can be either local or remote, but they are actively using the system.
* **Locally logged on users** are physically at the machine, working directly at its interface.
* **Remotely logged on users** are accessing the machine over the network, typically from another location using tools like RDP, VPN, or SSH.
