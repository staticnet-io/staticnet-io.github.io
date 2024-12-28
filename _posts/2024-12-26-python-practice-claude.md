---
title:   "Python Practice with Claude AI"
tagline: "Using AI as a Learning Tool"
excerpt: "Python Practice with Claude AI"
date: 2024-12-26
classes: wide
tags: ai python learning networking automation post
---
# Using AI as a Learning Tool

Recently I've been talking more with a few of the many AI chatbots currently available, for help and ideas on various topics. One of the things that came to mind was to ask for suggestions on how to brush up on Python skills relating to network engineering and network automation.  I've shared the responses I received from [Claude AI](https://claude.ai) below. In the near future I'll do a follow up post to cover the solutions I came up with for each exercise.

**NOTE:** solution links coming in a future update to this post.
{: .notice--info}

```
# Write a script that connects to multiple switches and verifies:
# - NTP servers are correctly configured
# - SNMP community strings match expected values
# - VLANs exist as expected
# This teaches you loops, conditionals, and basic network device interaction
```

```
# Create a script that:
# - Polls interface status every 5 minutes
# - Stores historical up/down status
# - Alerts (via email/Slack) if an interface flaps
# This introduces you to scheduling, file/database handling, and API integration
```

```
# Build a tool that:
# - Takes a "golden config" template
# - Compares actual device configs against it
# - Reports discrepancies
# Great for learning text parsing and comparison operations
```

```
# Develop a script that:
# - Reads changes from a CSV/Excel file
# - Validates the proposed changes
# - Implements them across multiple devices
# - Creates a backup and rollback plan
# Teaches error handling and safe deployment practices
```

Here's a starter template for the interface monitor to give you an idea:

```python
from netmiko import ConnectHandler
from datetime import datetime
import time
import json

def get_interface_status(device, interface):
    connection = ConnectHandler(**device)
    output = connection.send_command(f"show interface {interface}")
    connection.disconnect()
    # Parse the output to determine if interface is up/down
    return "up" if "up" in output.lower() else "down"

def monitor_interfaces(devices, check_interval=300):
    history = {}
    
    while True:
        for device in devices:
            current_status = get_interface_status(device, device['interface'])
            
            # Record status change
            if device['hostname'] not in history:
                history[device['hostname']] = []
            
            history[device['hostname']].append({
                'timestamp': datetime.now().isoformat(),
                'status': current_status
            })
            
            # Check for status change and alert if needed
            if len(history[device['hostname']]) >= 2:
                if history[device['hostname']][-1]['status'] != history[device['hostname']][-2]['status']:
                    print(f"Alert! Interface status changed on {device['hostname']}")
                    
        time.sleep(check_interval)

# Example usage
devices = [
    {
        'hostname': 'switch1',
        'device_type': 'cisco_ios',
        'ip': '192.168.1.1',
        'username': 'admin',
        'password': 'password',
        'interface': 'GigabitEthernet0/1'
    }
]
```



