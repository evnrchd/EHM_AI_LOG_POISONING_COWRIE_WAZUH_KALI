# EHM_AI_LOG_POISONING_COWRIE_WAZUH_KALI

**The most dangerous attacker doesn’t hide — they just alter your reality.**  
This lab weaponizes Cowrie logs to mislead Wazuh into trusting poisoned telemetry.  
A direct look at how detection pipelines collapse when an attacker seizes the upstream narrative.

---

## Objective
Demonstrate how a single crafted log can manipulate SIEM-driven detection, validate a custom Wazuh rule, and expose how fragile SOC visibility becomes when log integrity is compromised.

---

## Environment
- Wazuh Manager VM (log ingestion + detection engine)  
- Cowrie Honeypot VM (source of manipulated telemetry)  
- Kali Linux VM (attacker / log injector)  
- VirtualBox NAT Network  
- SSH protocol  

---

##  Setup Overview
- Deployed Cowrie and validated active JSON log generation  
- Connected Wazuh Agent and confirmed log forwarding  
- Created a custom Wazuh rule for Cowrie events  
- Injected crafted JSON directly into Cowrie logs  
- Restarted Wazuh Manager to load updated rules  
- Verified poisoning impact through alert generation  


---

##  Attack Simulation
- Created a forged Cowrie JSON record mimicking a real authentication event  
- Injected the poisoned log upstream into Cowrie’s log path  
- Wazuh decoded and accepted the forged event as legitimate  
- Triggered a real Wazuh alert for an incident that never occurred  
- Successfully demonstrated narrative manipulation inside a SIEM pipeline  

---

## Commands Used
# Cowrie Verification
-'ls -lh /home/cowrie/var/log/cowrie'
---
# Editing local Wazuh rules
-'sudo nano /var/ossec/etc/rules/local_rules.xml'
---
# Inject controlled poisoned JSON
-echo '{"event":"test_poison", "source":"cowrie"}' > /home/cowrie/var/log/cowrie/cowrie.json'
---
# Custom Wazuh rule
-<group name="cowrie">
<rule id="100501" level="5">
<source>log</source>
<decoded_as>json</decoded_as>
<match>cowrie</match>
<description>Cowrie honeypot event detected</description>
</rule>
</group>
---
# Restart Wazuh Manager
-'sudo systemctl restart wazuh-manager'
# Search for Cowrie-Related Alerts
-'grep cowrie /var/ossec/logs/alerts/alerts.log'
---
# Validated Wazuh dashbboard connectivity via Powershell
-'Test-NetConnection 10.0.0.166 -port 443'
---
## Network Diagram

```mermaid flowchart LR

subgraph Kali_Attacker ["Kali Linux VM (Attacker)"]
A1[Crafted JSON Poisoned Log<br>SSH Connectivity Tests]
end

subgraph Cowrie_VM ["Cowrie Honeypot VM"]
C1[JSON Log Generation<br>/var/log/cowrie/]
end

subgraph Wazuh_Manager ["Wazuh Manager VM"]
W1[Log Ingestion<br>Decoders + Rules Engine]
W2[Custom Rule 100501<br>Triggers Alert]
end

A1 -->|Injects Poisoned Log| C1
C1 -->|Forwards Telemetry| W1
W1 -->|Accepts & Parses Forged Event| W2
---
### Inject a controlled poisoned JSON event
```bash
echo '{"event":"test_poison","source":"cowrie","msg":"fake honeypot event"}' \
>> /home/cowrie/var/log/cowrie/cowrie.json
---
MITRE ATT&CK MAPPING
Technique
ID
How It Applies
Defense Evasion – Modify System Logs
T1070.002
The attacker injects forged JSON records into Cowrie’s log path to alter detection outcomes.
Defense Evasion – Obfuscated/Deceptive Logging
T1027
The poisoned event blends seamlessly with legitimate Cowrie telemetry to evade analyst suspicion.
Impact – Data Manipulation
T1565.002
Altering log data causes incorrect security conclusions and false incident generation.
Defense Evasion – Indicator Removal/Modification
T1070
By modifying upstream telemetry, an attacker can erase real events or fabricate fake ones.
Why this matters:
This lab proves that “log poisoning” is not sci-fi — it’s a legitimate ATT&CK-aligned threat that SIEM teams ignore at their own risk.
---
What I Learned:
	* How to operationalize Cowrie and Wazuh in a functioning detection environment
	*	How SIEMs trust upstream telemetry by default — and why that’s a weakness
	*	How a single poisoned log can alter SOC visibility and the entire incident storyline
	*	How to modify, load, and validate Wazuh detection rules
	*	How attackers can “win” without exploiting a box — they exploit trust
	*	Why modern SOC teams need integrity validation on their data sources
  * Relying on siem ai-log ingestion without validation is still extremely fragile

---
Final Notes:
This lab demonstrates a fundamental truth of cybersecurity:

Attacks don’t always break into systems — sometimes they break into narratives.

If an attacker controls your logs, they don’t need:
	*	persistence
	*	C2
	*	exploits
	*	privilege escalation

They only need your SIEM to believe a story that never happened.

