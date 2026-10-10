# Project 10: Linux Service Troubleshooting Lab

## 1. Project Overview

This project demonstrates how to troubleshoot Linux services using system administration commands. It covers checking service status, starting and stopping services, inspecting logs, identifying port conflicts, troubleshooting Nginx, and diagnosing common service failures.

The lab uses Amazon Linux 2023 and Nginx as an example service.

## 2. Objectives

- Understand Linux services and systemd.
- Check service status and startup configuration.
- Start, stop, restart, and reload services.
- Analyze service logs using journalctl.
- Troubleshoot failed services.
- Identify listening ports and port conflicts.
- Diagnose Nginx configuration and connectivity problems.
- Verify service recovery after troubleshooting.

## 3. Environment

- Operating system: Amazon Linux 2023
- Service manager: systemd
- Example service: Nginx
- Tools: `systemctl`, `journalctl`, `ps`, `ss`, `curl`, `grep`

> **Important:** Run these exercises on your own test EC2 instance or a lab machine. Avoid restarting or stopping production services without authorization. Sample outputs are illustrative.

## 4. Step 1: Check the Linux System

Check the operating system:

```bash
cat /etc/os-release
```

Example output:

```text
NAME="Amazon Linux"
VERSION="2023"
```

Check the current user:

```bash
whoami
```

Check the hostname:

```bash
hostname
```

Check the current uptime:

```bash
uptime
```

**Explanation:** These commands identify the system, the user running the commands, and the system's uptime and load averages.

## 5. Step 2: Understand systemd Services

List active services:

```bash
systemctl list-units --type=service --state=running
```

List services that have failed:

```bash
systemctl --failed
```

Check the systemd manager:

```bash
systemctl is-system-running
```

Possible output:

```text
running
```

The output may also be `degraded` if some units have failed.

**Explanation:** systemd manages services and other system units. Failed units should be investigated rather than automatically restarted without understanding the cause.

## 6. Step 3: Install Nginx

Refresh package metadata and install Nginx:

```bash
sudo dnf install nginx -y
```

Check the installed version:

```bash
nginx -v
```

Example output:

```text
nginx version: nginx/1.x.x
```

The exact version depends on the package available for your system.

## 7. Step 4: Check Nginx Service Status

Check the service status:

```bash
sudo systemctl status nginx
```

A healthy service should show information similar to:

```text
Active: active (running)
```

Check the service state directly:

```bash
systemctl is-active nginx
```

Expected output:

```text
active
```

Check whether it is configured to start automatically during boot:

```bash
systemctl is-enabled nginx
```

Possible output:

```text
disabled
```

or:

```text
enabled
```

**Explanation:** `is-active` checks the current state, while `is-enabled` checks whether the service is enabled for automatic startup.

## 8. Step 5: Start, Stop, Restart, and Reload Services

Start Nginx:

```bash
sudo systemctl start nginx
```

Stop Nginx:

```bash
sudo systemctl stop nginx
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

Reload Nginx configuration without a full restart:

```bash
sudo systemctl reload nginx
```

Check the status after an operation:

```bash
sudo systemctl status nginx --no-pager
```

**Explanation:**

- `start` starts a service.
- `stop` stops a service.
- `restart` stops and starts a service.
- `reload` asks a service to reload its configuration if supported.

Do not run these commands against important production services without approval.

## 9. Step 6: Enable Nginx at Boot

Enable automatic startup:

```bash
sudo systemctl enable nginx
```

Enable and start it in one operation:

```bash
sudo systemctl enable --now nginx
```

Verify:

```bash
systemctl is-enabled nginx
systemctl is-active nginx
```

Expected output:

```text
enabled
active
```

**Explanation:** A service may be running now but not enabled for the next boot. Check both states when diagnosing startup problems.

## 10. Step 7: Inspect Service Logs

View recent Nginx service logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

View logs from the current boot:

```bash
sudo journalctl -u nginx -b --no-pager
```

Follow new log messages:

```bash
sudo journalctl -u nginx -f
```

Press `Ctrl+C` to stop following the logs.

View recent system errors:

```bash
sudo journalctl -p err -b --no-pager
```

**Explanation:** Logs often reveal configuration errors, missing files, permission issues, failed dependencies, and port conflicts.

## 11. Step 8: Check Nginx Configuration

Test the Nginx configuration:

```bash
sudo nginx -t
```

Expected successful output:

```text
syntax is ok
test is successful
```

If the configuration is invalid, Nginx will report an error and usually identify the file and line involved.

Check the main configuration file:

```bash
sudo ls -l /etc/nginx/nginx.conf
```

Review it:

```bash
sudo less /etc/nginx/nginx.conf
```

Press `q` to exit `less`.

**Explanation:** Always test the configuration before restarting or reloading Nginx after configuration changes.

## 12. Step 9: Check Listening Ports

Check whether a service is listening on TCP port 80:

```bash
sudo ss -lntp | grep ':80'
```

Example output:

```text
LISTEN 0 511 0.0.0.0:80 0.0.0.0:*
```

Check the Nginx processes:

```bash
ps -ef | grep '[n]ginx'
```

Check all listening TCP ports:

```bash
sudo ss -lntp
```

**Explanation:** If a service fails to bind to its configured port, another process may already be using that port. Confirm which process owns the port before taking action.

## 13. Step 10: Test the Local Web Service

Make a local HTTP request:

```bash
curl -I http://127.0.0.1/
```

Example output:

```text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
```

Check the page content:

```bash
curl http://127.0.0.1/
```

If Nginx is running and serving its default page, the response will contain HTML.

**Explanation:** A successful local request confirms that the service is responding locally. It does not prove that the server is reachable from the internet.

## 14. Step 11: Troubleshoot a Service That Fails to Start

If Nginx fails to start, collect diagnostic information:

```bash
sudo systemctl status nginx --no-pager -l
sudo journalctl -u nginx -n 100 --no-pager
sudo nginx -t
sudo ss -lntp
```

Investigate the reported cause before changing anything.

Common causes include:

- Invalid configuration syntax.
- Another process using the required port.
- Missing files or directories.
- Incorrect file permissions.
- Missing service dependencies.
- Insufficient disk space.

Check disk space:

```bash
df -h
```

Check inode usage:

```bash
df -i
```

Check memory:

```bash
free -h
```

**Explanation:** A service failure can be caused by resources, configuration, permissions, or dependencies. Use evidence from the logs and commands to identify the actual cause.

## 15. Step 12: Practice a Safe Configuration Failure

Use a test configuration file instead of deliberately breaking the active Nginx configuration.

Create a temporary invalid configuration:

```bash
cat > /tmp/nginx-invalid.conf <<'EOF'
events {}
http {
    invalid_directive;
}
EOF
```

Test it:

```bash
sudo nginx -t -c /tmp/nginx-invalid.conf
```

Expected result: the test fails and reports an unknown or invalid directive.

Remove the temporary test file:

```bash
rm /tmp/nginx-invalid.conf
```

Check the actual service configuration:

```bash
sudo nginx -t
```

**Explanation:** This exercise demonstrates how configuration validation catches errors without disrupting the running service.

## 16. Step 13: Troubleshoot a Port Conflict

If Nginx reports that it cannot bind to port 80, inspect the listening ports:

```bash
sudo ss -lntp | grep ':80'
```

If `lsof` is installed, you can also run:

```bash
sudo lsof -iTCP:80 -sTCP:LISTEN
```

Possible findings include another web server or application already listening on port 80.

Do not kill a process immediately. First identify the application and determine whether it can safely be stopped or reconfigured.

After resolving the verified conflict, test and start Nginx:

```bash
sudo nginx -t
sudo systemctl start nginx
sudo systemctl status nginx --no-pager
```

## 17. Step 14: Troubleshoot External Connectivity

If Nginx works locally but cannot be accessed from another machine, check:

1. Nginx is running and listening on the expected interface and port.
2. The server firewall permits the intended traffic.
3. The AWS EC2 security group allows inbound TCP port 80 from the intended client network.
4. Network ACLs and routing are correct.
5. You are using the correct public IP address or DNS name.

Check the local response again:

```bash
curl -I http://127.0.0.1/
```

Check the listening port:

```bash
sudo ss -lntp | grep ':80'
```

**Explanation:** Local service health and external network reachability are different checks. Do not open ports to the entire internet unless the application genuinely requires it.

## 18. Step 15: Create a Troubleshooting Report

Create a report containing basic diagnostic information:

```bash
cat > service_report.sh <<'EOF'
#!/bin/bash

SERVICE="nginx"
REPORT="service_report.txt"

{
    echo "===================================="
    echo "Linux Service Troubleshooting Report"
    echo "===================================="
    echo "Date: $(date)"
    echo "Hostname: $(hostname)"
    echo

    echo "Service Status:"
    systemctl is-active "$SERVICE" 2>&1
    echo

    echo "Service Startup Configuration:"
    systemctl is-enabled "$SERVICE" 2>&1
    echo

    echo "Disk Usage:"
    df -h
    echo

    echo "Memory Usage:"
    free -h
    echo

    echo "Listening TCP Ports:"
    ss -lnt
    echo

    echo "Recent Service Logs:"
    journalctl -u "$SERVICE" -n 20 --no-pager 2>&1

} > "$REPORT"

echo "Report saved to $REPORT"
EOF
```

Make the script executable:

```bash
chmod +x service_report.sh
```

Run it:

```bash
./service_report.sh
```

Example output:

```text
Report saved to service_report.txt
```

View the report:

```bash
cat service_report.txt
```

Some checks may show errors if the service does not exist or the current user lacks permission to read its logs.

**Explanation:** A diagnostic report collects useful information into one file, making troubleshooting easier to document and review.

## 19. Step 16: Verify Recovery

After resolving the issue, run these verification commands:

```bash
sudo nginx -t
sudo systemctl is-active nginx
curl -I http://127.0.0.1/
```

Expected results:

- Nginx configuration test succeeds.
- The service state is `active`.
- The HTTP request receives a successful response, such as `200 OK`.

Record the problem, the evidence, the corrective action, and the final verification in your project notes.

## 20. Common Linux Troubleshooting Commands

| Command | Purpose |
|---|---|
| `systemctl status nginx` | Inspect service state |
| `systemctl restart nginx` | Restart a service |
| `systemctl enable nginx` | Enable startup at boot |
| `journalctl -u nginx` | Inspect service logs |
| `nginx -t` | Validate Nginx configuration |
| `ss -lntp` | Inspect listening TCP ports |
| `ps -ef` | View running processes |
| `df -h` | Check filesystem space |
| `df -i` | Check inode availability |
| `free -h` | Check memory usage |
| `curl -I URL` | Inspect HTTP response headers |

## 21. Project Outcome

After completing this project, I can:

- Inspect Linux service status.
- Start, stop, restart, and reload services.
- Configure service startup at boot.
- Analyze logs with journalctl.
- Validate Nginx configuration.
- Identify listening ports and potential conflicts.
- Troubleshoot local and external connectivity.
- Generate a basic service diagnostic report.
- Verify service recovery after troubleshooting.

## 22. Tools and Technologies

- Amazon Linux 2023
- systemd
- Nginx
- journalctl
- Bash scripting
- curl
- Linux networking commands
- AWS EC2 networking

## 23. Conclusion

This project demonstrates practical Linux service administration and troubleshooting skills. These skills are useful for Linux system administration, cloud operations, DevOps engineering, and production incident investigation.
