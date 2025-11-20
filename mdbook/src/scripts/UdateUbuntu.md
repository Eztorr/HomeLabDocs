# Udate Ubuntu Script

Script to update Ubuntu.

```
#!/bin/bash
# Ubuntu Update & Cleanup Script
# ------------------------------------------------------------
# This script updates an Ubuntu system, removes unused packages,
# and cleans old cache files.

set -e

echo "Updating package lists..."
sudo apt update

echo "Upgrading installed packages..."
sudo apt upgrade -y

echo "Performing full distribution upgrade..."
sudo apt full-upgrade -y

echo "Removing unused dependencies..."
sudo apt autoremove -y

echo "Cleaning local package cache..."
sudo apt autoclean -y
sudo apt clean -y

echo "System update and cleanup complete!"

```