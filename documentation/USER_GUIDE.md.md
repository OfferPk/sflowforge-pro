\# sFlowForge Pro \- User Guide & DevSecOps Integration Manual

\#\# 1\. Web Studio Quickstart

1\. Open \`source-code/index.html\` in any modern web browser.  
2\. Select an industry preset from the top dropdown:  
   \- \*\*Datacenter Spine 400G Multi-Gigabit Arista EOS (A+)\*\*: 400G spine deployment, 1:4000 sampling, 128B header capture for VXLAN inspection, 20s counter polling, and dual collector failover.  
   \- \*\*Cloud Edge Anti-DDoS Scrubbing Pipeline (FastNetMon) (A+)\*\*: Carrier-grade edge scrubbing on Cisco Nexus with 1:1000 sampling, 15s counter polling, and FastNetMon DDoS detection.  
   \- \*\*Vulnerable Legacy sFlow (Truncated 64B & Aliased 1:50k) (F)\*\*: Demonstrates failure modes from 64-byte truncated headers (breaking VXLAN), 1:50000 sampling (aliasing bursts), and single collector target.  
3\. Review the \*\*Sampling Calibration & Nyquist Calculator\*\*:  
   \- Change Port Link Speed (1G, 10G, 25G, 40G, 100G, 400G) to view the calibrated sampling rate recommendation.  
4\. Review the \*\*10-Point Zero-Trust Sampling Scorecard\*\* for telemetry hygiene.  
5\. Inspect the \*\*sFlow v5 XDR Binary Wireframe & Hex Dump\*\*:  
   \- Inspect byte offsets and ASCII decoding of the Datagram Header, Flow Sample (Tag 1), and Counter Sample (Tag 2).  
6\. Switch between synthesizer tabs to export configurations:  
   \- \*\*Arista EOS\*\*: Copy EOS switch configuration directives.  
   \- \*\*Cisco NX-OS\*\*: Copy Nexus 9000/3000 sFlow syntax.  
   \- \*\*Juniper Junos\*\*: Copy Junos QFX/EX \`set protocols sflow\` commands.  
   \- \*\*Host sFlow (hsflowd)\*\*: Copy Linux server \`hsflowd.conf\`.  
   \- \*\*FastNetMon\*\*: Copy DDoS engine sFlow configuration.  
   \- \*\*Python Test Client\*\*: Copy and execute the raw UDP packet sender.

\---

\#\# 2\. DevSecOps CLI Tool (\`sflowforge-cli.py\`)

The standalone Python CLI tool requires zero external dependencies and runs on Python 3.8+.

\#\#\# 2.1 Calculating Recommended Sampling Rates  
\`\`\`bash  
\# Calculate recommended sampling rate for 100G link  
python3 build/sflowforge-cli.py rate \-s 100

\# Calculate recommended sampling rate for 400G link with 800 samples/sec target  
python3 build/sflowforge-cli.py rate \-s 400 \-t 800  
\`\`\`

\#\#\# 2.2 Linting sFlow Configuration Presets  
\`\`\`bash  
python3 build/sflowforge-cli.py lint \-p presets/datacenter-spine-400g-arista.json  
\`\`\`  
Output:  
\`\`\`  
\[LINT PASS\] Preset: Datacenter Spine 400G Multi-Gigabit Arista EOS  
  Link Speed: 400G, Sampling Rate: 1:4000  
  Header Bytes: 128B, Polling Interval: 20s  
  Agent IP: 10.200.4.1, Collector: sflow-collector-01.corp.internal:6343  
\`\`\`

\#\#\# 2.3 Running CI/CD Pipeline Telemetry Audits  
\`\`\`bash  
\# Clean production configuration \-\> Exits code 0  
python3 build/sflowforge-cli.py audit \-p presets/datacenter-spine-400g-arista.json \--strict

\# Insecure/fragile configuration \-\> Exits code 1 (Pipeline Fails)  
python3 build/sflowforge-cli.py audit \-p presets/vulnerable-legacy-sflow-v5.json \--strict  
\`\`\`

\#\#\# 2.4 Synthesizing Vendor Configurations  
\`\`\`bash  
\# Generate Arista EOS configuration  
python3 build/sflowforge-cli.py synthesize \-p presets/datacenter-spine-400g-arista.json \-f arista

\# Generate Cisco NX-OS configuration  
python3 build/sflowforge-cli.py synthesize \-p presets/datacenter-spine-400g-arista.json \-f cisco  
\`\`\`

\---

\#\# 3\. GitHub Actions CI/CD Pipeline Template

Add this workflow to your NetDevOps or Ansible infrastructure repository (\`.github/workflows/sflow-telemetry-gate.yml\`):

\`\`\`yaml  
name: Zero-Trust sFlow Telemetry Policy CI/CD Gate

on:  
  pull\_request:  
    paths:  
      \- 'sflow/\*\*'  
      \- 'arista/\*\*'  
      \- 'nexus/\*\*'  
      \- 'telemetry/\*\*'

jobs:  
  sflow-audit:  
    runs-on: ubuntu-latest  
    steps:  
      \- name: Checkout Code  
        uses: actions/checkout@v4

      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'

      \- name: Audit sFlow Configuration (Strict Gate)  
        run: |  
          python3 build/sflowforge-cli.py audit \-p presets/datacenter-spine-400g-arista.json \--strict  
\`\`\`  
