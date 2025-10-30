# Azure VWAN BGP and IPsec Discovery Commands

This short script set helps Azure engineers and network architects quickly identify BGP and IPsec configuration details for a Virtual WAN (VWAN) VPN Gateway.  
It’s especially useful when configuring or validating on-premises VPN devices (Cisco, Palo Alto, FortiGate, etc.) that connect to Azure VWAN hubs.

The output provides:
- The Azure VPN Gateway ASN (Autonomous System Number)
- The public IP addresses used for IPsec tunnels (Instance 0 and 1)
- The Azure-side BGP peer IPs used for route exchange inside the tunnel

---

## ☁️ Running in Azure Cloud Shell

You can run these commands directly in Azure Cloud Shell, which is a browser-based terminal built into the Azure Portal.  
Cloud Shell comes preinstalled with the Azure CLI, PowerShell, and common tools such as Git, Python, and text editors. It automatically authenticates with your logged-in Azure account, making it a secure and convenient environment for running management commands — no local setup required.

To launch Cloud Shell:
1. Sign in to the Azure Portal.
2. Click the Cloud Shell icon (▣) in the top navigation bar.
3. Select Bash (recommended for these examples).

> **Important:** Make sure you are in the correct Azure subscription context before running any of these commands.  
> You can verify or set the subscription using:
```bash
az account show
az account set --subscription "<your-subscription-name-or-id>"
```

---

## 🧭 List all gateways in your resource group

```bash
az network vpn-gateway list   -g lab-svh-intra   -o table
```

This shows you the exact VPN gateway name (e.g., `sechub1-vpngw`) to use in the next step.

---

## 📡 Show ASN + BGP + Public IPs (per instance)

You can either set the gateway name using the output above or paste it inline. Both options are shown below.

### Option A — Set a variable from the first command’s output
```bash
# Manually set the gateway name you saw in the list (recommended)
GW="sechub1-vpngw"   # <-- replace with your gateway name from the list

# Now run the summary query
az network vpn-gateway show   -g lab-svh-intra   -n "$GW"   --query "{AzureASN:bgpSettings.asn, Peers:bgpSettings.bgpPeeringAddresses[].{Instance:ipconfigurationId, PublicIP:tunnelIpAddresses[0], AzureBGP:defaultBgpIpAddresses[0]}}"   -o json > ${GW}-bgp-summary.json
```

> Tip: If you only have one gateway and want to auto-pick it:
```bash
GW=$(az network vpn-gateway list -g lab-svh-intra --query "[0].name" -o tsv)
echo "Using gateway: $GW"
```

### Option B — Use the name inline
```bash
az network vpn-gateway show   -g lab-svh-intra   -n sechub1-vpngw   --query "{AzureASN:bgpSettings.asn, Peers:bgpSettings.bgpPeeringAddresses[].{Instance:ipconfigurationId, PublicIP:tunnelIpAddresses[0], AzureBGP:defaultBgpIpAddresses[0]}}"   -o json > sechub1-vpngw-bgp-summary.json
```

This command outputs your Azure VPN Gateway’s ASN, BGP peer IPs, and public IP addresses for both active instances.  
It also saves the data to a JSON file for documentation or GitHub reference.

---

### 💡 Example Output (simplified)
```json
{
  "AzureASN": 65515,
  "Peers": [
    {
      "Instance": "Instance0",
      "PublicIP": "172.175.208.71",
      "AzureBGP": "192.168.1.12"
    },
    {
      "Instance": "Instance1",
      "PublicIP": "20.57.93.58",
      "AzureBGP": "192.168.1.13"
    }
  ]
}
```

This output provides all the IPs required for your on-premises VPN configuration and BGP peering setup.

---

### 🔒 Notes
- Always double-check before publishing IP information publicly.  
- Works for both Standard and Secured VWAN hubs.  
- JSON output can be versioned or pushed to GitHub for quick collaboration or automation workflows.
