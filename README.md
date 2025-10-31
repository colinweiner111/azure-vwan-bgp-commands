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

This command lists all VPN gateways within your resource group.  
You’ll need the **Name** value (e.g., `sechub1-vpngw`) from this output to use in the next command.

```bash
az network vpn-gateway list --resource-group <ResourceGroupName> -o table
```

```bash
az network vpn-gateway list --resource-group lab-svh-intra -o table
```

### 💡 Example Output
```
EnableBgpRouteTranslationForNat    IsRoutingPreferenceInternet    Location    Name             ProvisioningState    ResourceGroup    VpnGatewayScaleUnit
---------------------------------  -----------------------------  ----------  ----------------  -------------------  ----------------  -------------------
False                              False                          eastus2     sechub1-vpngw     Succeeded            lab-svh-intra     1
False                              False                          eastus2     sechub2-vpngw     Succeeded            lab-svh-intra     1
```

From the example above, you can use either **sechub1-vpngw** or **sechub2-vpngw** in the next command.

---

## 📡 Show ASN + BGP + Public IPs (per instance)

This command write show the BGP ASN, BGP peer IPs, and Public IPs per instance to a json file.  
You’ll need the **Name** value (e.g., `sechub1-vpngw`) from this output to use in the next command.

```bash
az network vpn-gateway show --resource-group <ResourceGroupName> --name <HubName> --query "{AzureASN:bgpSettings.asn, Peers:bgpSettings.bgpPeeringAddresses[].{Instance:ipconfigurationId, PublicIP:tunnelIpAddresses[0], AzureBGP:defaultBgpIpAddresses[0]}}" -o json > sechub1-bgp-summary.json
```

```bash
az network vpn-gateway show --resource-group lab-svh-intra -n sechub1-vpngw --query "{AzureASN:bgpSettings.asn, Peers:bgpSettings.bgpPeeringAddresses[].{Instance:ipconfigurationId, PublicIP:tunnelIpAddresses[0], AzureBGP:defaultBgpIpAddresses[0]}}" -o json > sechub1-bgp-summary.json
```

After running the command above, the output will be saved as **sechub1-bgp-summary.json**.  
You can use `code sechub1-bgp-summary.json` in **Cloud Shell** to open it directly in the editor, or download the file to your local machine to view it.  
The file contains data similar to the example output shown below.

---

### 💡 Example Output (in file)
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
- TBD

