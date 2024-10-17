
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


## To Create Bulk User


```markdown
# Bulk User Creation Script

This bash script allows you to create multiple users on a Linux system in bulk. It reads a list of usernames and passwords from a file and creates corresponding user accounts, setting up specific directories and permissions for each user.

## Usage

``` bash
#!/bin/bash
 
# Check if the input file is provided
if [ $# -ne 1 ]; then
  echo "Usage: $0 <userfile>"
  exit 1
fi
 
# Read the input file
input_file="$1"
 
# Loop through each line in the file
while IFS=' ' read -r user_name password; do
  # Create the user's home directory
  mkdir -p "/home/$user_name/uploads"

  # Create the user with the specified shell and home directory
  useradd -s /bin/bash -d "/home/$user_name/uploads" "$user_name"

  chmod 770 /home/$user_name/uploads
  # Set the user's password
  echo "$user_name:$password" | chpasswd
 
  echo "User $user_name created with home directory /home/$user_name/uploads"
done < "$input_file"


```

```bash
./create_users.sh <userfile>
```

- `<userfile>`: A text file containing usernames and passwords, with each line formatted as follows:
  ```
  username password
  ```

### Example

Create a file named `userlist.txt` with the following content:

```
user1 password1
user2 password2
```

Then, run the script:

```bash
./create_users.sh userlist.txt
```

## Script Functionality

1. **Input File Check**: The script expects one argument – the path to the user file. If not provided, it will display a usage message and exit.
   
2. **Reading User File**: The script reads the file line by line. Each line should contain a username and a password separated by a space.

3. **User Creation**:
   - For each user, it creates a home directory inside `/home/<username>/uploads`.
   - Adds the user with `/bin/bash` as their default shell and sets their home directory to `/home/<username>/uploads`.
   - Sets appropriate permissions (`770`) on the user's home directory.
   
4. **Password Setup**: The script sets the provided password for each user using the `chpasswd` command.

5. **Confirmation**: For each user, a message confirming the creation of the user and their home directory is displayed.

## Requirements

- Bash shell
- Root or sudo privileges to create users

## Important Notes

- Ensure the input file is properly formatted with a space between the username and password.
- The script must be run as root or using `sudo` to create users and set passwords.
- Be cautious when handling passwords in plaintext files for security purposes.

## Example Output

```
User user1 created with home directory /home/user1/uploads
User user2 created with home directory /home/user2/uploads
```

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
```

Feel free to modify it as needed!


---

## Conclusion

This guide helps you configure an SFTP environment where you can either have separate directories for SSH and SFTP or use the same home directory for both.
