# Azure VWAN BGP Commands

This short script identifies BGP and IPsec configuration IP addresses for a Virtual WAN (VWAN) VPN Gateway.  
It’s especially useful when configuring or validating on‑premises VPN devices (Cisco, Palo Alto, FortiGate, etc.) that connect to Azure VWAN hubs.

The output provides:
- The Azure VPN Gateway ASN (Autonomous System Number)
- The public IP addresses used for IPsec tunnels (Instance 0 and 1)
- The Azure‑side BGP peer IPs used for route exchange inside the tunnel

---

## ☁️ Running in Azure Cloud Shell (Bash)

You can run these commands directly in **Azure Cloud Shell** — a browser‑based terminal in the Azure Portal with Azure CLI preinstalled and already authenticated to your account.  
To launch Cloud Shell: sign in to the Azure Portal → click the **Cloud Shell** icon (▣) → **select Bash**.

> Make sure you’re in the **correct subscription** before running the commands:
```bash
az account show
az account set --subscription "<your-subscription-name-or-id>"
```

---

## 🧭 List all gateways in your resource group

```bash
az network vpn-gateway list -g lab-svh-intra -o table
```

---

## 📡 Show ASN + BGP + Public IPs (per instance)

```bash
az network vpn-gateway show -g lab-svh-intra -n sechub1-vpngw --query "{AzureASN:bgpSettings.asn, Peers:bgpSettings.bgpPeeringAddresses[].{Instance:ipconfigurationId, PublicIP:tunnelIpAddresses[0], AzureBGP:defaultBgpIpAddresses[0]}}" -o json > sechub1-bgp-summary.json
```

This command outputs your Azure VPN Gateway’s ASN, BGP peer IPs, and public IP addresses for both active instances, and saves them to a JSON file for documentation or GitHub reference.

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

This output provides all the IPs required for your on‑premises VPN configuration and BGP peering setup.

---

### 🔒 Notes
- Double‑check before publishing IP information publicly.  
- Works for both Standard and Secured VWAN hubs.  
- JSON output can be versioned or pushed to GitHub for collaboration or automation.
