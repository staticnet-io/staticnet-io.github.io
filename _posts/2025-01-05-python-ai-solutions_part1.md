---
title:  "Python Network Automation Practice with AI - Solutions Part 1"
date: 2025-01-05
last_modified_at: 
tagline: "Part 1: Configuration Validator" # overrides excerpt for page tagline display
excerpt: "Part 1: Configuration Validator" # SEO, etc
layout: 
classes: wide
tags:
---


# Solutions to AI Python Practice Exercises - Part 1: Network Configuration Verification

Welcome to the first installment of the "Python Practice with AI - Solutions" series! This post explores a practical network automation solution that demonstrates key Python concepts including network device interaction, file handling, and regex pattern matching. Whether you're new to network automation or looking to enhance Python skills, this exercise provides valuable hands-on experience with real-world networking tasks.

## What You'll Learn
- Working with Python's `re` module for parsing network configurations
- Handling device credentials securely
- Processing structured data (JSON and CSV)
- Implementing error handling for network operations
- Writing maintainable network automation code

## Prerequisites
Before starting, ensure you have the following:
```bash
# Required Python packages
netmiko>=4.1.0  # For network device interaction
```

You can install the required package using:
```bash
pip install netmiko
```

**NOTE:** I **always** prefer using Python version management and virtual environments. This is not covered in this blog post, but I'll try to put something together in a future post. For now, I would suggest to look into something like [asdf](https://asdf-vm.com/){:target="_blank"} and Python's [venv](https://docs.python.org/3/library/venv.html){:target="_blank"}.
{: .notice--info}

## The Challenge
Write a script that connects to multiple network switches and verifies:
- NTP server configurations
- SNMP community strings
- VLAN configurations

This exercise teaches practical implementation of loops, conditionals, and network device interaction in a real-world context.

**Link to the original post:** [Python Network Automation Practice with AI]({% post_url 2024-12-26-python-network-auto-practice-ai %}){:target="_blank"}
{: .notice--warning}

## Solution Overview

### 1. Configuration Files

First, let's look at our configuration files. We'll use two files to separate our device inventory from our expected configurations:

#### config.json
This file defines our expected network configurations:
```json
{
    "ntp_servers": [
        "10.0.0.1",
        "10.0.0.2"
    ],
    "snmp_communities": ["public", "private"],
    "vlans": [1, 10, 20, 30, 40]
}
```

#### inventory.csv
This file contains our device inventory:
```
hostname,device_type,port
172.16.101.250,cisco_ios,22
172.16.102.250,cisco_ios,22
172.16.103.250,cisco_ios,22
172.16.104.250,cisco_ios,22
```

### 2. Python Implementation
The following sections can be assembled into a single script.

- #### Imports
```python
from netmiko import ConnectHandler
import re
import sys
import csv
from pathlib import Path
from getpass import getpass
```

- #### `load_inventory` function:
```python
def load_inventory(inventory_file, username, password, enable_secret=None):
    """
    Load device inventory from CSV file and add credentials.
    Expected CSV format:
    hostname,device_type,port
    192.168.1.1,cisco_ios,22
    """
    devices = []
    
    try:
        with open(inventory_file, 'r') as f:
            reader = csv.DictReader(f)
            for row in reader:
                device = {
                    'host': row['hostname'],
                    'device_type': row['device_type'],
                    'username': username,
                    'password': password,
                    'port': int(row.get('port', 22)),  # Default to port 22 if not specified
                }
                
                # Only add secret if it's provided
                if enable_secret:
                    device['secret'] = enable_secret
                    
                devices.append(device)
    except FileNotFoundError:
        print(f"Error: Inventory file '{inventory_file}' not found.")
        sys.exit(1)
    except KeyError as e:
        print(f"Error: Missing required column in CSV file: {e}")
        sys.exit(1)
        
    return devices
```
- #### `load_config` function:
```python
def load_config(config_file):
    """
    Load expected configuration from JSON file.
    """
    import json
    
    try:
        with open(config_file, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"Error: Configuration file '{config_file}' not found.")
        sys.exit(1)
    except json.JSONDecodeError as e:
        print(f"Error: Invalid JSON in configuration file: {e}")
        sys.exit(1)
```

- #### Configuration verification functions:
- `verify_ntp_config`:

```python
def verify_ntp_config(conn, expected_servers):
    """
    Verify NTP server configuration.
    Handles NTP servers specified as IP addresses, hostnames, or FQDNs.
    """
    output = conn.send_command('show running-config | include ntp server')
    
    # Match any server specification after "ntp server"
    configured_servers = re.findall(r'ntp server\s+(\S+)', output)
    
    # Convert all servers to lowercase for case-insensitive comparison
    configured_servers = [server.lower() for server in configured_servers]
    expected_servers_lower = [server.lower() for server in expected_servers]
    
    results = {
        'status': all(server in configured_servers for server in expected_servers_lower),
        'configured': configured_servers,
        'expected': expected_servers
    }
    return results
```

   - `verify_snmp_config`:

```python
def verify_snmp_config(conn, expected_communities):
    """Verify SNMP community string configuration."""
    output = conn.send_command('show running-config | include snmp-server community')
    configured_communities = re.findall(r'snmp-server community (\S+)', output)
    
    results = {
        'status': all(comm in configured_communities for comm in expected_communities),
        'configured': configured_communities,
        'expected': expected_communities
    }
    return results
```
   - `verify_vlans`:

```python
def verify_vlans(conn, expected_vlans):
    """Verify VLAN configuration."""
    output = conn.send_command('show vlan brief')
    configured_vlans = [int(vlan) for vlan in re.findall(r'^\d+', output, re.MULTILINE)]
    
    results = {
        'status': all(vlan in configured_vlans for vlan in expected_vlans),
        'configured': configured_vlans,
        'expected': expected_vlans
    }
    return results
```

- The `check_device_config` and `print_results` functions:

```python
def check_device_config(device, expected_config):
    """Check configuration for a single device."""
    try:
        with ConnectHandler(**device) as conn:
            hostname = conn.find_prompt().rstrip('#>')
            
            results = {
                'hostname': hostname,
                'ip': device['host'],
                'ntp': verify_ntp_config(conn, expected_config['ntp_servers']),
                'snmp': verify_snmp_config(conn, expected_config['snmp_communities']),
                'vlans': verify_vlans(conn, expected_config['vlans']),
                'status': 'success'
            }
    except Exception as e:
        results = {
            'hostname': 'Unknown',
            'ip': device['host'],
            'status': 'failed',
            'error': str(e)
        }
    
    return results

def print_results(results):
    """Print verification results in a formatted way."""
    for device in results:
        print(f"\n{'='*50}")
        print(f"Device: {device['hostname']} ({device['ip']})")
        print(f"{'='*50}")
        
        if device['status'] == 'failed':
            print(f"ERROR: {device['error']}")
            continue
            
        # Print NTP results
        print("\nNTP Configuration:")
        print(f"Status: {'✓' if device['ntp']['status'] else '✗'}")
        print(f"Configured servers: {', '.join(device['ntp']['configured'])}")
        print(f"Expected servers: {', '.join(device['ntp']['expected'])}")
        
        # Print SNMP results
        print("\nSNMP Configuration:")
        print(f"Status: {'✓' if device['snmp']['status'] else '✗'}")
        print(f"Configured communities: {', '.join(device['snmp']['configured'])}")
        print(f"Expected communities: {', '.join(device['snmp']['expected'])}")
        
        # Print VLAN results
        print("\nVLAN Configuration:")
        print(f"Status: {'✓' if device['vlans']['status'] else '✗'}")
        print(f"Configured VLANs: {', '.join(map(str, device['vlans']['configured']))}")
        print(f"Expected VLANs: {', '.join(map(str, device['vlans']['expected']))}")
```

- The `main` function:

```python
def main():
    import argparse
    
    parser = argparse.ArgumentParser(description='Network Configuration Verification Tool')
    parser.add_argument('-i', '--inventory', required=True, help='Path to inventory CSV file')
    parser.add_argument('-c', '--config', required=True, help='Path to expected configuration JSON file')
    parser.add_argument('-u', '--username', help='Username for device authentication')
    args = parser.parse_args()
    
    # Get credentials interactively
    username = args.username or input("Enter username: ")
    password = getpass("Enter password: ")
    enable_secret = getpass("Enter enable secret (press Enter if not needed): ") or None
    
    # Load inventory and configuration
    devices = load_inventory(args.inventory, username, password, enable_secret)
    expected_config = load_config(args.config)
    
    # Process devices sequentially
    results = []
    for device in devices:
        result = check_device_config(device, expected_config)
        results.append(result)
    
    print_results(results)

if __name__ == '__main__':
    main()
```

---
---


### Understanding the Code

Let's break down some key aspects of the solution:

1. The `load_inventory` function:
   - Reads device information from a CSV file
   - Adds authentication credentials to each device
   - Handles basic error cases like missing files or columns
   - Returns a list of device dictionaries ready for Netmiko

2. Configuration verification functions:
   - `verify_ntp_config`: Uses regex to extract NTP servers from the running config
   - `verify_snmp_config`: Checks SNMP community strings
   - `verify_vlans`: Parses VLAN information from 'show vlan brief'
   
3. Error handling approach:
   - Each verification function returns a consistent dictionary structure
   - Failed connections are caught and reported clearly
   - Results include both expected and actual configurations

### Example Output
Here's what the verification results look like:

```
==================================================
Device: SW-CORE-01 (172.16.101.250)
==================================================

NTP Configuration:
Status: ✓
Configured servers: 10.0.0.1, 10.0.0.2
Expected servers: 10.0.0.1, 10.0.0.2

SNMP Configuration:
Status: ✗
Configured communities: public
Expected communities: public, private

VLAN Configuration:
Status: ✓
Configured VLANs: 1, 10, 20, 30, 40
Expected VLANs: 1, 10, 20, 30, 40
```

## Common Issues and Troubleshooting

1. Connection Timeouts
- Verify network connectivity to the device
- Check if the device is accepting SSH connections on the specified port
- Ensure no firewalls are blocking the connection

2. Authentication Failures
- Verify username and password are correct
- Check if enable secret is required
- Ensure the user has sufficient privileges

3. Configuration Parsing Errors
- Different device vendors may format output differently
- Some commands may take longer to execute
- Verify command output format matches expectations

## Security Considerations

1. Credential Management
- Never hardcode credentials in scripts
- Use environment variables or secure credential storage
- Implement role-based access control

2. SNMP Security
- Use SNMPv3 where possible
- Implement access control lists

## Next Steps Ideas
- Modify the script to support different network vendors
- Add support for configuration backup
- Implement parallel device verification for larger networks
- Add reporting capabilities (CSV, HTML, etc.)

Remember to check out the [previous post]({% post_url 2024-12-26-python-network-auto-practice-ai %}) for more Python practice exercises!

## Feedback
Found a bug or have a suggestion? Feel free to leave a comment below

Happy coding!