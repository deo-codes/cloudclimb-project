# CloudClimb Project 01 — Troubleshooting Log

## Week 02 — SSH Connectivity Troubleshooting

**Project:** CloudClimb Project 01 — Azure Infrastructure  
**Task:** Task 10 — Troubleshooting Challenge  
**Environment:** Azure Ubuntu VM, Virtual Network, Network Security Group  
**Affected Resource:** `vm-app-dev-eastus`  
**Status:** Resolved

### 1. Symptom

During Week 2, I experienced difficulty establishing an SSH connection from my local computer to the Azure Ubuntu virtual machine.

The VM was deployed in the Application Subnet, but remote SSH access was unsuccessful.

### 2. What I Checked

I investigated the SSH connection path:

Local Computer → Internet → VM Public IP → Network Security Group → Network Interface → Ubuntu VM

I reviewed the following:

- Virtual machine availability and public IP configuration.
- Application subnet and network interface association.
- Network Security Group inbound security rules.
- TCP port 22 access restrictions.
- The source public IP address permitted by the SSH rule.
- SSH connection settings, including the username and private key.

### 3. Root Cause

The SSH inbound rule in `nsg-app-dev-eastus` did not permit the correct public source IP address for the computer/network being used.

Because the NSG restricted SSH traffic to a specific source address, the connection was blocked when the permitted address did not match the actual source.

### 4. Resolution

I corrected the SSH inbound security rule in Azure.

The updated configuration allowed:

- **Protocol:** TCP
- **Destination port:** 22
- **Source:** Authorized public IP address
- **Action:** Allow

I maintained source-IP restrictions rather than opening SSH access to the entire internet.

### 5. Verification

After updating the NSG configuration, I successfully connected to the Ubuntu VM using SSH key-based authentication.

I verified that the SSH session opened and that I could execute Linux commands on the VM.

**Result:** Remote administrative access was restored without rebuilding the VM.

### 6. Lessons Learned

- Network Security Groups control inbound and outbound traffic to Azure resources.
- SSH access can fail even when a virtual machine is running.
- The administrator's public IP address is different from the VM's private IP address.
- Restricting TCP port 22 to authorized source IPs improves security.
- Troubleshooting should follow the connection path before attempting to rebuild infrastructure.
- Successful SSH access should be verified after changing network security rules.

### 7. Future Improvements

- Maintain a documented SSH access procedure.
- Verify the administrator's current public IP before changing NSG rules.
- Review NSG rules regularly to remove unnecessary access.
- Consider Azure Bastion or other private administrative access methods for future production-style environments.

---

**Final Outcome:** Diagnosed and resolved an SSH connectivity issue caused by an incorrect NSG source IP configuration.

# Week 3 — Troubleshooting Challenge

**Project:** CloudClimb Project 01 — Azure Infrastructure  
**Incident Type:** PostgreSQL Authentication Failure  
**Environment:** Azure Ubuntu VM, Docker, Memos, Azure PostgreSQL Flexible Server  
**Status:** Resolved

## 1. Incident Summary

During a controlled troubleshooting simulation, I deliberately configured an incorrect PostgreSQL database password in the Memos application's environment file.

After recreating the Docker container, Memos failed to initialize its PostgreSQL connection.

## 2. Error Identified

I investigated the failure using:

`sudo docker logs --tail 40 memos`

The logs reported:

```text
FATAL: password authentication failed for user "cloudadmin"
SQLSTATE 28P01
```

This indicated that PostgreSQL rejected the credentials supplied by Memos.

## 3. Root Cause

The `MEMOS_DSN` connection string in `/home/azureuser/.config/memos/postgres.env` contained an incorrect database password.

The failure was caused by invalid database credentials, not a confirmed network connectivity issue.

## 4. Resolution

I restored the working configuration from a protected backup:

```bash
cp -p ~/.config/memos/postgres.env.backup ~/.config/memos/postgres.env
```

I then recreated the Docker container so it would load the restored credentials.

```bash
sudo docker stop memos
sudo docker rm memos
```

The container was started again with the protected environment file:

```bash
sudo docker run -d \
  --name memos \
  --restart unless-stopped \
  -p 5230:5230 \
  --env-file /home/azureuser/.config/memos/postgres.env \
  -v /home/azureuser/.memos:/var/opt/memos \
  neosmemo/memos:stable
```

## 5. Verification

After restoring the configuration, I verified that:

- The Memos Docker container was running.
- Docker logs reported that Memos started successfully.
- The database driver was PostgreSQL.
- The application loaded successfully in the browser.
- The original PostgreSQL integration test memo was still available.

**Result:** Application connectivity was restored without losing existing data.

## 6. Lessons Learned

- Docker containers must be recreated to apply changes made to environment files supplied at creation.
- PostgreSQL error `28P01` indicates invalid password authentication.
- Application logs are essential for identifying startup failures.
- Protected configuration backups can reduce recovery time.
- Recovery must be verified at both the container and application levels.
- Database passwords and connection strings must never be committed to GitHub.

## 7. Architecture Considerations

The application runs on an Azure Ubuntu VM in East US, while the temporary PostgreSQL Flexible Server runs in Central US.

The database uses a restricted public endpoint and TLS-encrypted connections because the originally planned private PostgreSQL deployment in East US was unavailable.

Private PostgreSQL networking remains a future improvement.

---

**Final Outcome:** Successfully reproduced, diagnosed, and resolved a PostgreSQL authentication failure while preserving application data.