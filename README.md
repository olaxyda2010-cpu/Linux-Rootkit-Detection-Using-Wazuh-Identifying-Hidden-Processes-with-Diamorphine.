# Linux-Rootkit-Detection-Using-Wazuh-Identifying-Hidden-Processes-with-Diamorphine.

# Wazuh Rootcheck: Hidden Process Detection Using Diamorphine

## 1. Project Overview

This project demonstrates hidden-process detection on an Ubuntu endpoint using the Diamorphine Linux kernel rootkit and Wazuh Rootcheck.

The objective is to investigate how Diamorphine can hide selected processes from standard Linux process inspection commands and how Wazuh Rootcheck may detect suspicious activity.

## 2. Objectives

- Configure Wazuh Rootcheck to scan every 120 seconds.
- Investigate the Diamorphine kernel module.
- Examine Linux process visibility using `ps`.
- Investigate the `rsyslogd` hidden-process scenario.
- Review potential Rootcheck alerts in the Wazuh dashboard.

## 3. Lab Environment

| Component | Details |
|---|---|
| Endpoint | Ubuntu Linux |
| Hostname | `vtcsec` |
| Kernel Version | `4.15.0-142-generic` |
| Rootkit | Diamorphine |
| Monitoring Agent | Wazuh Agent |
| Detection Component | Wazuh Rootcheck |
| Virtualization | VirtualBox |

## 4. Wazuh Rootcheck Configuration

The Wazuh agent configuration file is:

```bash
/var/ossec/etc/ossec.conf
```

The lab guide specifies a Rootcheck scan frequency of 120 seconds.

Relevant configuration:

```xml
<rootcheck>
  <disabled>no</disabled>
  <check_files>yes</check_files>
  <check_trojans>yes</check_trojans>
  <check_dev>yes</check_dev>
  <check_sys>yes</check_sys>
  <check_pids>yes</check_pids>
  <check_ports>yes</check_ports>
  <check_if>yes</check_if>
  <frequency>120</frequency>
  <rootkit_files>etc/shared/rootkit_files.txt</rootkit_files>
  <rootkit_trojans>etc/shared/rootkit_trojans.txt</rootkit_trojans>
  <skip_nfs>yes</skip_nfs>
</rootcheck>
```
<img width="730" height="459" alt="image" src="https://github.com/user-attachments/assets/a2ae4205-3cff-4af2-ac39-2549d77c8657" />

After modifying the configuration, the lab guide instructs the analyst to restart the agent:

```bash
sudo systemctl restart wazuh-agent
```

## 5.Attack emulation
On the Ubuntu Endpoint, do the following:

## Fetch the Diamorphine rootkit source code from GitHub:

```bash
   
https://github.com/m0nad/Diamorphine 
```

## Install make tool:
 
 ```bash
sudo apt install make

```


##  Navigate to the Diamorphine directory and compile the source code:
```bash
cd Diamorphine
make
```
## Load the rootkit kernel module:
```bash
insmod diamorphine.ko
```
Note: The kernel-level rootkit “Diamorphine” is now installed on the Ubuntu endpoint.
Depending on the environment, the module sometimes fails to load or function properly. If
you receive the error insmod: ERROR: could not insert module diamorphine.ko: Invalid
parameters in the last step, you can restart the Linux endpoint and try again. Sometimes it
takes several tries for it to work.

 
## Get a PID:
```bash
 ps aux | head
```

## Run the kill signal 63 with the PID process running on the Ubuntu endpoint you got above.
This unhides the Diamorphine rootkit. By default, Diamorphine hides itself so we don’t detect
it by running the lsmod command. Try it out:

```bash

lsmod | grep diamorphine
kill -63 <RANDOME-PID>
lsmod | grep diamorphine
```
Output:
```bash

diamorphine 16384 0
```

**Note**: When using these last commands, you can expect an empty output. In the case of
Diamorphine, any kill signal 63 sent to any process whether it exists or not, toggles the
Diamorphine kernel module to hide or unhide.


## When using this last command, you can expect an empty output:
```bash
kill -31 <PID_OF_RSYSLOGD>
ps auxw | grep rsyslog | grep -v grep
```
<img width="731" height="464" alt="image" src="https://github.com/user-attachments/assets/5784797c-0e8c-42b8-8f28-8a36ade43c87" />
<img width="737" height="463" alt="image" src="https://github.com/user-attachments/assets/849a0310-ad55-45cd-9ea1-e05791b821ee" />


## 6.The next rootcheck scan will run and alert us about the rsyslogd process which was hidden
with the Diamorphine rootkit.

Visualize the alerts:
You can visualize the alert data in the Wazuh dashboard. To do this, go to the Threat
Hunting module
<img width="1366" height="646" alt="image" src="https://github.com/user-attachments/assets/2979c172-4744-4024-bb56-8d67d3500268" />


The detection workflow is:

1. Inspect the target process.
2. Observe the process visibility change during the authorized lab test.
3. Allow the configured Rootcheck scan to run.
4. Review the resulting Wazuh alert.
5. Record the alert details and supporting evidence.

**Detection status:** A corresponding Wazuh alert  was verified in the evidence collected for this report.

## 7. Results

| Test | Result |
|---|---|
| Diamorphine source files | Found |
| Compiled module file | Found |
| Kernel metadata version | Matches running kernel |
| Diamorphine visible in `lsmod` | No output |
| Hidden `rsyslogd` process |  verified |
| Wazuh Rootcheck alert |  verified |

## 8. Security Implications

Kernel-level rootkits can undermine the reliability of ordinary host inspection tools.

Security recommendations:

- Monitor unexpected kernel module activity.
- Investigate discrepancies in process visibility.
- Review Wazuh Rootcheck configuration and scan results.
- Correlate endpoint findings with kernel logs and other trusted telemetry.
- Keep the operating system and monitoring tools maintained.
- Conduct rootkit experiments in isolated virtual machines.

## 9. Conclusion

This project investigates the use of Diamorphine to demonstrate hidden-process behavior and the role of Wazuh Rootcheck in identifying suspicious activity.

The investigation confirmed the presence of Diamorphine source files and a compiled module. The module metadata also matched the running kernel version.

 The collected evidence  confirm that Diamorphine was active, that `rsyslogd` was successfully hidden, and that Wazuh generated the expected alert.

