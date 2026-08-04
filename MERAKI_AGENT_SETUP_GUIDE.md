# Meraki AI Agent Setup Guide

Complete step-by-step guide to create an AI Agent that generates Meraki network health reports with logs, remediations, alerts summary, inventory, and recommendations.

**Generated:** August 4, 2026
**Document Version:** 1.0

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Setup Instructions](#setup-instructions)
5. [Implementation Guide](#implementation-guide)
6. [Report Components](#report-components)
7. [Testing & Validation](#testing--validation)
8. [Troubleshooting](#troubleshooting)
9. [Advanced Features](#advanced-features)

---

## Overview

This guide helps you build an AI Agent that connects to your Meraki Dashboard and generates comprehensive network health reports including:

- **Network Health**: Overall status, device availability, connection quality
- **Logs & Events**: Recent system events and security incidents (48-hour window)
- **Remediations & Alerts**: Alert configuration and remediation action history
- **Inventory Report**: Device counts by type, firmware versions, license usage
- **Recommendations**: Performance optimization, security improvements, capacity planning

### Recommended Approach

**Best Option: Meraki Magic MCP + Claude (AI-Native)**
- Access to ~804 Meraki API endpoints
- Automatic tool discovery
- No manual API management
- Zero configuration after setup

**Alternative: Python Script**
- Self-contained, no Claude dependency
- Full programmatic control
- Can be scheduled with cron or task scheduler

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI Agent (Claude)                            │
│                  (Report Generator)                             │
└───────────────────┬─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│            Meraki Magic MCP Server                              │
│  (804 API Endpoints + Dynamic Tool Discovery)                   │
└───────────────────┬─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│            Meraki Dashboard API (v1)                            │
│  • Organization Management                                      │
│  • Network Monitoring                                           │
│  • Device Health & Status                                       │
│  • Alerts & Events                                              │
│  • Security & Appliance                                         │
│  • Wireless & Switch Management                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

### Required
- [ ] Cisco Meraki Dashboard account with admin access
- [ ] Python 3.13 or later installed on your machine
- [ ] Meraki API key (generated from dashboard)
- [ ] Meraki Organization ID
- [ ] Git installed
- [ ] Claude Desktop (for MCP approach) or just Python (for script approach)

### Optional
- Docker (for containerized deployment)
- VS Code or any Python IDE
- Jupyter notebook support (for analysis)

### Estimated Time
- **MCP Setup**: 15-20 minutes
- **Python Script Setup**: 10-15 minutes
- **First Report Generation**: 5-10 minutes

---

## Setup Instructions

### Step 1: Get Your Meraki API Credentials

**This is required for both approaches.**

#### 1.1 Enable Dashboard API Access

1. Log in to your **Meraki Dashboard** (https://dashboard.meraki.com)
2. Navigate to **Organization → Settings**
3. Scroll to **Dashboard API access**
4. Click **Enable API access**

#### 1.2 Generate API Key

1. Go to **Organization → Settings → Dashboard API access**
2. Under **API keys**, click **Generate new API key**
3. Copy the generated API key and **save it securely** (you won't see it again)
4. **⚠️ IMPORTANT**: Never commit this to version control or share it

```
Example API Key format:
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
(This is a fake example)
```

#### 1.3 Get Your Organization ID

1. Still in **Organization → Settings**
2. Look for **Organization ID** at the top of the page
3. Copy this value (format: `123456`)

```
Example Organization ID:
123456
(This is a fake example)
```

#### 1.4 Verify Your Credentials (Optional)

Test your credentials by running:

```bash
curl -X GET "https://api.meraki.com/api/v1/organizations" \
  -H "X-Cisco-Meraki-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

You should get a JSON response with your organization details.

---

### Step 2A: Setup with Meraki Magic MCP (Recommended)

**Follow this section if you want to use Claude Desktop with MCP.**

#### 2A.1 Clone the Repository

Open your terminal and run:

```bash
# Navigate to where you want to store the project
cd ~/projects  # or any directory you prefer

# Clone the Meraki Magic MCP repository
git clone https://github.com/CiscoDevNet/meraki-magic-mcp-community.git

# Navigate into the directory
cd meraki-magic-mcp-community
```

#### 2A.2 Create Virtual Environment

**macOS/Linux:**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows (PowerShell):**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Windows (Command Prompt):**
```cmd
python -m venv .venv
.venv\Scripts\activate.bat
```

#### 2A.3 Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Expected output should show successful installation of:
- `meraki` SDK
- `fastmcp`
- `pydantic`
- Other dependencies

#### 2A.4 Configure Environment Variables

1. Copy the example file:
```bash
cp .env-example .env
```

2. Edit `.env` with your credentials:

```env
# Your Meraki API Key (from Step 1.2)
MERAKI_API_KEY="your_actual_api_key_here"

# Your Organization ID (from Step 1.3)
MERAKI_ORG_ID="your_org_id_here"

# API Base URL (default for global, change if in specific region)
MERAKI_BASE_URL="https://api.meraki.com/api/v1"

# Performance Settings (optional)
ENABLE_CACHING=true
CACHE_TTL_SECONDS=300
READ_ONLY_MODE=true  # Prevents accidental modifications
```

**💡 Regional Endpoints** (if applicable):
- **Canada**: `https://api.meraki.ca/api/v1`
- **China**: `https://api.meraki.cn/api/v1`
- **India**: `https://api.meraki.in/api/v1`
- **US Federal**: `https://api.gov-meraki.com/api/v1`

#### 2A.5 Configure Claude Desktop

**macOS:**

1. Open the Claude config file:
```bash
open ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

2. Or use your preferred editor:
```bash
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Windows:**

1. Open File Explorer and navigate to:
```
%APPDATA%\Claude\claude_desktop_config.json
```

2. Or use command line:
```powershell
notepad $env:APPDATA\Claude\claude_desktop_config.json
```

3. Add the following configuration to the `mcpServers` section:

**macOS Example:**
```json
{
  "mcpServers": {
    "Meraki_Magic_MCP": {
      "command": "/Users/YourUsername/projects/meraki-magic-mcp-community/.venv/bin/fastmcp",
      "args": [
        "run",
        "-t", "stdio",
        "/Users/YourUsername/projects/meraki-magic-mcp-community/meraki-mcp-dynamic.py"
      ]
    }
  }
}
```

**Windows Example:**
```json
{
  "mcpServers": {
    "Meraki_Magic_MCP": {
      "command": "C:/Users/YourUsername/projects/meraki-magic-mcp-community/.venv/Scripts/fastmcp.exe",
      "args": [
        "run",
        "-t", "stdio",
        "C:/Users/YourUsername/projects/meraki-magic-mcp-community/meraki-mcp-dynamic.py"
      ]
    }
  }
}
```

⚠️ **Important**: Replace `/path/to/` with your actual installation path. Windows users: use forward slashes `/` in the path.

#### 2A.6 Restart Claude Desktop

1. **Completely quit** Claude Desktop (don't just minimize it)
2. Reopen Claude Desktop
3. You should now have access to Meraki tools

#### 2A.7 Verify Setup

In Claude Desktop, ask:
```
What MCP servers are available?
```

You should see "Meraki_Magic_MCP" in the response.

---

### Step 2B: Setup with Python Script (Alternative)

**Follow this section if you prefer a standalone Python approach.**

#### 2B.1 Create Project Directory

```bash
mkdir meraki-report-agent
cd meraki-report-agent
```

#### 2B.2 Create Virtual Environment

**macOS/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

**Windows (PowerShell):**
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

#### 2B.3 Create requirements.txt

Create a file named `requirements.txt`:

```
meraki>=4.3.0
pydantic>=2.0
python-dateutil>=2.8
requests>=2.31.0
python-dotenv>=1.0.0
```

#### 2B.4 Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 2B.5 Create .env File

Create a file named `.env`:

```env
MERAKI_API_KEY=your_actual_api_key_here
MERAKI_ORG_ID=your_org_id_here
MERAKI_BASE_URL=https://api.meraki.com/api/v1
```

#### 2B.6 Create the Report Agent Script

Create a file named `report_generator.py` with the content shown in the [Python Script Section](#python-script-reportgeneratorpy) below.

#### 2B.7 Run the Script

```bash
python report_generator.py
```

This will generate a report and save it to `meraki_report.json`.

---

## Python Script (report_generator.py)

```python
import os
import json
import meraki
from datetime import datetime, timedelta
from typing import Dict, List, Any
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

class MerakiReportAgent:
    """
    AI Agent for generating Meraki network health reports.
    Collects data on network health, events, alerts, inventory, and recommendations.
    """
    
    def __init__(self, api_key: str = None, org_id: str = None):
        """Initialize the Meraki Report Agent"""
        self.api_key = api_key or os.getenv('MERAKI_API_KEY')
        self.org_id = org_id or os.getenv('MERAKI_ORG_ID')
        
        if not self.api_key or not self.org_id:
            raise ValueError("MERAKI_API_KEY and MERAKI_ORG_ID environment variables are required")
        
        # Initialize Meraki Dashboard API with smart flow rate limiting
        self.dashboard = meraki.DashboardAPI(
            api_key=self.api_key,
            smart_flow_enabled=True,
            print_console=False,  # Suppress console output
            suppress_logging=True
        )
        
        self.report = {
            "generated_at": datetime.now().isoformat(),
            "org_id": self.org_id,
            "sections": {}
        }
    
    def get_network_health(self) -> Dict[str, Any]:
        """Collect network health metrics"""
        print("📊 Collecting network health data...")
        
        try:
            org_status = self.dashboard.organizations.getOrganizationStatus(
                organizationId=self.org_id
            )
            
            networks = self.dashboard.organizations.getOrganizationNetworks(
                organizationId=self.org_id
            )
            
            devices_by_network = {}
            devices_online = 0
            devices_offline = 0
            
            for network in networks:
                try:
                    devices = self.dashboard.networks.getNetworkDevices(
                        networkId=network['id']
                    )
                    devices_by_network[network['name']] = len(devices)
                    
                    for device in devices:
                        if device.get('status') == 'online':
                            devices_online += 1
                        else:
                            devices_offline += 1
                except Exception as e:
                    print(f"  ⚠️  Error: {e}")
            
            total_devices = devices_online + devices_offline
            uptime_percentage = (devices_online / total_devices * 100) if total_devices > 0 else 0
            
            health_data = {
                "organization": {
                    "id": self.org_id,
                    "status": org_status[0].get('status', 'unknown') if org_status else 'unknown'
                },
                "network_count": len(networks),
                "device_summary": {
                    "total": total_devices,
                    "online": devices_online,
                    "offline": devices_offline,
                    "uptime_percentage": round(uptime_percentage, 2)
                },
                "devices_by_network": devices_by_network,
                "timestamp": datetime.now().isoformat()
            }
            
            print(f"✅ Network Health: {devices_online}/{total_devices} online ({uptime_percentage:.1f}%)")
            return health_data
            
        except Exception as e:
            print(f"❌ Error: {e}")
            return {"error": str(e)}
    
    def get_recent_events(self, hours: int = 48) -> Dict[str, Any]:
        """Get recent events and logs"""
        print(f"📋 Collecting events (last {hours} hours)...")
        
        try:
            networks = self.dashboard.organizations.getOrganizationNetworks(
                organizationId=self.org_id
            )
            
            all_events = []
            event_types = {}
            critical_events = []
            
            for network in networks:
                try:
                    events = self.dashboard.networks.getNetworkEvents(
                        networkId=network['id'],
                        timespan=hours * 3600
                    )
                    
                    for event in events:
                        event['network_name'] = network['name']
                        all_events.append(event)
                        
                        event_type = event.get('eventType', 'unknown')
                        event_types[event_type] = event_types.get(event_type, 0) + 1
                        
                        if any(keyword in str(event).lower() for keyword in ['down', 'error', 'failed', 'critical']):
                            critical_events.append(event)
                
                except Exception as e:
                    print(f"  ⚠️  Error: {e}")
            
            all_events.sort(key=lambda x: x.get('occurredAt', ''), reverse=True)
            critical_events.sort(key=lambda x: x.get('occurredAt', ''), reverse=True)
            
            events_data = {
                "total_events": len(all_events),
                "event_types": event_types,
                "critical_events_count": len(critical_events),
                "recent_events": all_events[:50],
                "critical_events": critical_events[:10],
                "timestamp": datetime.now().isoformat()
            }
            
            print(f"✅ Events: {len(all_events)} total, {len(critical_events)} critical")
            return events_data
            
        except Exception as e:
            print(f"❌ Error: {e}")
            return {"error": str(e)}
    
    def get_alerts_summary(self) -> Dict[str, Any]:
        """Get alerts configuration"""
        print("🔔 Collecting alerts...")
        
        try:
            networks = self.dashboard.organizations.getOrganizationNetworks(
                organizationId=self.org_id
            )
            
            alerts_summary = []
            
            for network in networks:
                try:
                    settings = self.dashboard.networks.getNetworkAlertsSettings(
                        networkId=network['id']
                    )
                    
                    try:
                        alert_history = self.dashboard.networks.getNetworkAlertsHistory(
                            networkId=network['id'],
                            timespan=7 * 24 * 3600
                        )
                    except:
                        alert_history = []
                    
                    alerts_summary.append({
                        "network_id": network['id'],
                        "network_name": network['name'],
                        "alert_settings": settings,
                        "alert_history": alert_history
                    })
                
                except Exception as e:
                    print(f"  ⚠️  Error: {e}")
            
            alerts_data = {
                "networks_with_alerts": len(alerts_summary),
                "alert_configurations": alerts_summary,
                "timestamp": datetime.now().isoformat()
            }
            
            print(f"✅ Alerts: {len(alerts_summary)} networks")
            return alerts_data
            
        except Exception as e:
            print(f"❌ Error: {e}")
            return {"error": str(e)}
    
    def get_inventory(self) -> Dict[str, Any]:
        """Get device inventory"""
        print("📦 Collecting inventory...")
        
        try:
            inventory = self.dashboard.organizations.getOrganizationInventory(
                organizationId=self.org_id
            )
            
            license_info = self.dashboard.organizations.getOrganizationLicenseState(
                organizationId=self.org_id
            )
            
            device_types = {}
            firmware_versions = {}
            
            for device in inventory:
                device_type = device.get('model', 'unknown')
                device_types[device_type] = device_types.get(device_type, 0) + 1
                
                firmware = device.get('firmwareVersion', 'unknown')
                firmware_key = f"{device_type}_{firmware}"
                firmware_versions[firmware_key] = firmware_versions.get(firmware_key, 0) + 1
            
            inventory_data = {
                "total_devices": len(inventory),
                "device_types": device_types,
                "firmware_versions": firmware_versions,
                "licenses": license_info,
                "devices": inventory[:100],
                "timestamp": datetime.now().isoformat()
            }
            
            print(f"✅ Inventory: {len(inventory)} devices")
            return inventory_data
            
        except Exception as e:
            print(f"❌ Error: {e}")
            return {"error": str(e)}
    
    def get_recommendations(self) -> Dict[str, Any]:
        """Generate recommendations"""
        print("💡 Generating recommendations...")
        
        recommendations = {
            "performance": [],
            "security": [],
            "capacity": [],
            "general": []
        }
        
        try:
            networks = self.dashboard.organizations.getOrganizationNetworks(
                organizationId=self.org_id
            )
            
            inventory = self.dashboard.organizations.getOrganizationInventory(
                organizationId=self.org_id
            )
            
            device_types = {}
            offline_devices = []
            
            for device in inventory:
                device_type = device.get('model', 'unknown')
                device_types[device_type] = device_types.get(device_type, 0) + 1
                
                if device.get('status') == 'offline':
                    offline_devices.append(device)
            
            if len(networks) == 0:
                recommendations["general"].append("Create your first network")
            
            for device_type, count in device_types.items():
                if count > 50:
                    recommendations["capacity"].append(
                        f"Consider segmenting {device_type} ({count} devices)"
                    )
            
            recommendations["security"].append("Enable 2FA for admin accounts")
            recommendations["security"].append("Review admin access permissions")
            recommendations["security"].append("Enable API access logging")
            
            recommendations["general"].append("Setup webhook alerts")
            recommendations["general"].append("Document network topology")
            recommendations["general"].append("Establish firmware update schedule")
            
            if len(offline_devices) > 0:
                recommendations["general"].append(
                    f"Investigate {len(offline_devices)} offline devices"
                )
            
            recommendations["capacity"].append(
                f"Current capacity: {len(inventory)} devices across {len(networks)} networks"
            )
            
            recommendations_data = {
                "performance": recommendations["performance"],
                "security": recommendations["security"],
                "capacity": recommendations["capacity"],
                "general": recommendations["general"],
                "timestamp": datetime.now().isoformat()
            }
            
            print(f"✅ Generated recommendations")
            return recommendations_data
            
        except Exception as e:
            print(f"❌ Error: {e}")
            return {"error": str(e)}
    
    def generate_report(self) -> Dict[str, Any]:
        """Generate complete report"""
        print("\n" + "="*60)
        print("🚀 MERAKI NETWORK HEALTH REPORT GENERATOR")
        print("="*60 + "\n")
        
        self.report["sections"]["network_health"] = self.get_network_health()
        self.report["sections"]["events"] = self.get_recent_events()
        self.report["sections"]["alerts"] = self.get_alerts_summary()
        self.report["sections"]["inventory"] = self.get_inventory()
        self.report["sections"]["recommendations"] = self.get_recommendations()
        
        print("\n" + "="*60)
        print("✅ REPORT GENERATION COMPLETE")
        print("="*60 + "\n")
        
        return self.report
    
    def save_report(self, filename: str = "meraki_report.json") -> str:
        """Save report to JSON file"""
        with open(filename, 'w') as f:
            json.dump(self.report, f, indent=2)
        print(f"💾 Report saved to: {filename}")
        return filename
    
    def print_summary(self):
        """Print report summary"""
        print("\n" + "="*60)
        print("REPORT SUMMARY")
        print("="*60 + "\n")
        
        if "network_health" in self.report["sections"]:
            health = self.report["sections"]["network_health"]
            if "device_summary" in health:
                summary = health["device_summary"]
                print(f"📊 NETWORK HEALTH")
                print(f"   Total Devices: {summary['total']}")
                print(f"   Online: {summary['online']}")
                print(f"   Offline: {summary['offline']}")
                print(f"   Uptime: {summary['uptime_percentage']}%\n")
        
        if "events" in self.report["sections"]:
            events = self.report["sections"]["events"]
            if "total_events" in events:
                print(f"📋 EVENTS & LOGS")
                print(f"   Total Events (48h): {events.get('total_events', 0)}")
                print(f"   Critical Events: {events.get('critical_events_count', 0)}\n")
        
        if "inventory" in self.report["sections"]:
            inventory = self.report["sections"]["inventory"]
            if "total_devices" in inventory:
                print(f"📦 INVENTORY")
                print(f"   Total Devices: {inventory.get('total_devices', 0)}")
                print(f"   Device Types: {len(inventory.get('device_types', {}))}\n")
        
        print("="*60 + "\n")


def main():
    """Main entry point"""
    try:
        agent = MerakiReportAgent()
        report = agent.generate_report()
        agent.print_summary()
        agent.save_report("meraki_report.json")
        print("✅ Report generation successful!")
        return 0
    except Exception as e:
        print(f"❌ Error: {e}")
        return 1


if __name__ == "__main__":
    exit(main())
```

---

## Testing & Validation

### Step 1: Verify API Connectivity

```bash
curl -X GET "https://api.meraki.com/api/v1/organizations" \
  -H "X-Cisco-Meraki-API-Key: YOUR_API_KEY"
```

### Step 2: Run Script

```bash
python report_generator.py
```

### Step 3: Validate Report

Open `meraki_report.json` and verify all sections are populated.

---

## Troubleshooting

### Invalid API Key
- Check API key in .env
- Verify API access is enabled
- Regenerate a new API key

### Organization Not Found
- Verify org ID (numbers only)
- Check admin access
- Confirm correct format

### Rate Limiting
- Script has auto-retry
- Enable caching: `ENABLE_CACHING=true`

### MCP Not Showing in Claude
- Restart Claude completely
- Check JSON syntax in claude_desktop_config.json
- Verify paths are correct

---

## Summary Checklist

- [ ] Get Meraki API Key
- [ ] Get Organization ID
- [ ] Test API connectivity
- [ ] Choose approach (MCP or Python)
- [ ] Complete setup
- [ ] Generate first report
- [ ] Verify all data is present

---

## Resources

- **Meraki API**: https://developer.cisco.com/meraki/api-v1/
- **MCP Server**: https://github.com/CiscoDevNet/meraki-magic-mcp-community
- **Python SDK**: https://github.com/meraki/dashboard-api-python
- **Community**: https://community.meraki.com

---

**Happy Reporting! 🚀**
