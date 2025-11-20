# Script for cleaning container before making an LXC Template

This script updates all system packages to the latest version, cleans up unused packages and cached data, removes existing SSH host keys (for regeneration on next boot), and clears the machine-id. Make sure to reboot the system or manually config new ssh keys to allow remote access.  
Works on Ubuntu

```
#!/bin/bash
# ---------------------------------------------------------------------------
# System Update, Cleanup, and Host Identity Reset Script
# ---------------------------------------------------------------------------
# This script:
#   1. Updates all system packages to the latest version
#   2. Cleans up unused packages and cached data
#   3. Removes existing SSH host keys (for regeneration on next boot)
#   4. Clears the machine-id (useful for cloning or creating templates)
#
#  WARNING:
#   - Removing SSH host keys means you’ll need to regenerate them manually
#     (usually done automatically at next reboot or with `dpkg-reconfigure openssh-server`).
#   - Clearing /etc/machine-id resets the system identity. Do this only on
#     cloned images or VM templates.
# ---------------------------------------------------------------------------

set -e 

echo "=== Step 1: Updating and upgrading all packages ==="
sudo apt -y update && sudo apt -y dist-upgrade

echo "=== Step 2: Cleaning up APT cache and unused packages ==="
sudo apt clean
sudo apt autoremove -y

echo "=== Step 3: Removing existing SSH host keys ==="
# Remove all SSH host key files so new ones are generated later
sudo rm -f /etc/ssh/ssh_host_*

echo "=== Step 4: Clearing machine-id ==="
# Truncate the file to zero bytes to reset the machine's unique ID
sudo truncate -s 0 /etc/machine-id

echo "=== Completed successfully! ==="
echo "  If this system will continue to be used, reboot it now to regenerate new SSH keys."
```