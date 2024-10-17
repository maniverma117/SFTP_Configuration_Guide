
# SFTP Configuration Guide

This guide outlines the steps to set up SFTP with SSH, where users can either have different landing directories for SFTP and SSH or the same landing directory for both.

## Prerequisites

- Root or sudo access to the server.
- SSH and SFTP access should be installed and properly configured on the system.

---

## 1. Setting Up SFTP with Different Landing Directory for SFTP and SSH

### Overview

In this setup, users have separate landing directories for SSH and SFTP. The SFTP directory is `/sftp`, and users' individual directories are set within `/sftp/<username>`.

### Steps

1. **Create SFTP Group**

   Create a new group for SFTP users:

   ```bash
   sudo groupadd sftpusers
   ```

2. **Create Base Directory for SFTP**

   Create a base directory for SFTP access:

   ```bash
   sudo mkdir /sftp
   sudo mkdir /sftp/user1
   sudo mkdir /sftp/user2
   sudo chown root:root /sftp
   sudo chmod 755 /sftp
   ```

   The `/sftp` directory and subdirectories must be owned by `root` to comply with the `ChrootDirectory` requirements.

3. **Create User-Specific Directories**

   Create individual user directories within `/sftp` and assign ownership:

   ```bash
   sudo mkdir /sftp/user1/files
   sudo mkdir /sftp/user2/files
   sudo chown user1:user1 /sftp/user1/files
   sudo chown user2:user2 /sftp/user2/files
   ```

   The `files` subdirectory is where each user will have full access.

4. **Assign Users to the SFTP Group**

   Add users to the `sftpusers` group:

   ```bash
   sudo usermod -aG sftpusers user1
   sudo usermod -aG sftpusers user2
   ```

5. **Edit the SSH Configuration**

   Modify the SSH configuration to define different landing directories for SFTP access:

   ```bash
   sudo vim /etc/ssh/sshd_config
   ```

   Add the following configuration at the end of the file:

   ```bash
   Match Group sftpusers
       ChrootDirectory /sftp/%u/
       ForceCommand internal-sftp
       AllowTcpForwarding no
       X11Forwarding no
   ```

   - `ChrootDirectory /sftp/%u/`: Restricts each user to their respective `/sftp/<username>` directory.
   - `ForceCommand internal-sftp`: Forces the use of the internal SFTP server.

6. **Restart the SSH Service**

   Apply the changes by restarting the SSH service:

   ```bash
   sudo systemctl restart sshd
   ```

---

## 2. Setting Up SFTP with the Same Landing Directory for SFTP and SSH

### Overview

In this configuration, the same home directory is used for both SFTP and SSH. The SFTP user is restricted to their home directory using a `ChrootDirectory`.

### Steps

1. **Edit the SSH Configuration**

   Modify the SSH configuration to set up the same landing directory for SFTP and SSH:

   ```bash
   sudo vim /etc/ssh/sshd_config
   ```

   Add the following configuration:

   ```bash
   Match User %u
       ChrootDirectory /home/%u
       AllowTcpForwarding no
       X11Forwarding no
       ForceCommand internal-sftp
   ```

   - `ChrootDirectory /home/%u`: Restricts each user to their respective `/home/<username>` directory.
   - `ForceCommand internal-sftp`: Forces the use of the internal SFTP server.


2. **Restart the SSH Service**

   Apply the changes by restarting the SSH service:

   ```bash
   sudo systemctl restart sshd
   ```

---

## Notes

- **Security Considerations**: Ensure that the SFTP user's home directories have the proper ownership (`root:root`) and permissions (`755`) for the `ChrootDirectory` to function correctly.
  
- **Testing**: After setting up, test the configuration by logging in with an SFTP client and ensuring that users are restricted to their designated directories and cannot navigate outside them.

---

## Troubleshooting

- If users encounter "Permission denied" errors while logging in via SFTP, check:
  - Ownership of the `ChrootDirectory` (`root:root`).
  - The permissions of the user's `files` directory should allow write access (`chown user:user` for their files).
  
- If changes don't take effect, ensure that the SSH daemon has been restarted:

  ```bash
  sudo systemctl restart sshd
  ```

- Review SSH logs for errors:

  ```bash
  sudo tail -f /var/log/auth.log
  ```

---

## Conclusion

This guide helps you configure an SFTP environment where you can either have separate directories for SSH and SFTP or use the same home directory for both.
