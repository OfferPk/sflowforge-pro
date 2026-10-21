/\*\*  
 \* sFlowForge Pro \- Application State Orchestrator & UI Controller  
 \*/

(function () {  
  'use strict';

  const PRESETS \= {  
    'datacenter-spine-400g-arista': {  
      name: "Datacenter Spine 400G Multi-Gigabit Arista EOS",  
      targetPlatform: "arista-eos",  
      linkSpeedGbps: 400,  
      samplingRate: 4000,  
      headerBytes: 128,  
      pollingIntervalSeconds: 20,  
      counters64Bit: true,  
      dropPacketAccounting: true,  
      agentSourceIp: "10.200.4.1",  
      agentSourceInterface: "Loopback0",  
      subAgentId: 1,  
      collectorHost: "sflow-collector-01.corp.internal",  
      secondaryCollectorHost: "sflow-collector-02.corp.internal",  
      port: 6343,  
      collectorBufferMb: 32,  
      extendedSwitchData: true,  
      extendedRouterData: true,  
      direction: "both"  
    },  
    'cloud-ddos-fastnetmon': {  
      name: "Cloud Edge Anti-DDoS Scrubbing Pipeline (FastNetMon)",  
      targetPlatform: "cisco-nxos",  
      linkSpeedGbps: 100,  
      samplingRate: 1000,  
      headerBytes: 128,  
      pollingIntervalSeconds: 15,  
      counters64Bit: true,  
      dropPacketAccounting: true,  
      agentSourceIp: "172.24.0.1",  
      agentSourceInterface: "Loopback101",  
      subAgentId: 101,  
      collectorHost: "fastnetmon-cluster.cloud.internal",  
      secondaryCollectorHost: "fastnetmon-backup.cloud.internal",  
      port: 6343,  
      collectorBufferMb: 64,  
      extendedSwitchData: true,  
      extendedRouterData: true,  
      direction: "both"  
    },  
    'vulnerable-legacy-sflow-v5': {  
      name: "Vulnerable Legacy sFlow (Truncated 64B & Aliased 1:50k)",  
      targetPlatform: "juniper-junos",  
      linkSpeedGbps: 10,  
      samplingRate: 50000,  
      headerBytes: 64,  
      pollingIntervalSeconds: 120,  
      counters64Bit: false,  
      dropPacketAccounting: false,  
      agentSourceIp: "",  
      agentSourceInterface: "ge-0/0/1",  
      subAgentId: 0,  
      collectorHost: "192.168.1.200",  
      secondaryCollectorHost: "",  
      port: 6343,  
      collectorBufferMb: 2,  
      extendedSwitchData: false,  
      extendedRouterData: false,  
      direction: "ingress"  
    }  
  };

  let activeVendorTab \= 'tab-arista';

  // DOM Elements  
  const elPresetSelect \= document.getElementById('preset-select');  
  const elBtnResetPreset \= document.getElementById('btn-reset-preset');  
  const elBtnExportJson \= document.getElementById('btn-export-json');

  const elLinkSpeed \= document.getElementById('link-speed');  
  const elSamplingRate \= document.getElementById('sampling-rate');  
  const elCalcSpeedText \= document.getElementById('calc-speed-text');  
  const elCalcRecRate \= document.getElementById('calc-rec-rate');  
  const elHeaderBytes \= document.getElementById('header-bytes');  
  const elCounterPolling \= document.getElementById('counter-polling');

  const elAgentIp \= document.getElementById('agent-ip');  
  const elSubAgentId \= document.getElementById('subagent-id');  
  const elPrimaryCollector \= document.getElementById('primary-collector');  
  const elCollectorPort \= document.getElementById('collector-port');  
  const elSecondaryCollector \= document.getElementById('secondary-collector');

  const elChk64BitCounters \= document.getElementById('chk-64bit-counters');  
  const elChkDropAccounting \= document.getElementById('chk-drop-accounting');  
  const elChkExtendedSwitch \= document.getElementById('chk-extended-switch');  
  const elChkExtendedRouter \= document.getElementById('chk-extended-router');  
  const elCollectorBuffer \= document.getElementById('collector-buffer');

  const elScorecardGrade \= document.getElementById('scorecard-grade');  
  const elScoreFill \= document.getElementById('score-fill');  
  const elScorePoints \= document.getElementById('score-points');  
  const elScorePercentage \= document.getElementById('score-percentage');  
  const elPassedCount \= document.getElementById('passed-count');  
  const elWarnCount \= document.getElementById('warn-count');  
  const elFailCount \= document.getElementById('fail-count');  
  const elRubricChecklist \= document.getElementById('rubric-checklist');

  const elTotalPacketSize \= document.getElementById('total-packet-size');  
  const elHexDumpCode \= document.getElementById('hex-dump-code');  
  const elBtnCopyHexDump \= document.getElementById('btn-copy-hexdump');

  const elVendorSynthesizerCode \= document.getElementById('vendor-synthesizer-code');  
  const elBtnCopyConfig \= document.getElementById('btn-copy-config');  
  const elBtnDownloadConfig \= document.getElementById('btn-download-config');

  function getCurrentConfig() {  
    return {  
      linkSpeedGbps: parseFloat(elLinkSpeed.value) || 100,  
      samplingRate: parseInt(elSamplingRate.value, 10\) || 1000,  
      headerBytes: parseInt(elHeaderBytes.value, 10\) || 128,  
      pollingIntervalSeconds: parseInt(elCounterPolling.value, 10\) || 20,  
      agentSourceIp: elAgentIp.value.trim(),  
      agentSourceInterface: "Loopback0",  
      subAgentId: parseInt(elSubAgentId.value, 10\) || 1,  
      collectorHost: elPrimaryCollector.value.trim(),  
      secondaryCollectorHost: elSecondaryCollector.value.trim(),  
      port: parseInt(elCollectorPort.value, 10\) || 6343,  
      counters64Bit: elChk64BitCounters.checked,  
      dropPacketAccounting: elChkDropAccounting.checked,  
      extendedSwitchData: elChkExtendedSwitch.checked,  
      extendedRouterData: elChkExtendedRouter.checked,  
      collectorBufferMb: parseInt(elCollectorBuffer.value, 10\) || 32,  
      direction: "both"  
    };  
  }

  function applyConfig(cfg) {  
    if (\!cfg) return;  
    elLinkSpeed.value \= cfg.linkSpeedGbps || 400;  
    elSamplingRate.value \= cfg.samplingRate || 4000;  
    elHeaderBytes.value \= cfg.headerBytes || 128;  
    elCounterPolling.value \= cfg.pollingIntervalSeconds || 20;  
    elAgentIp.value \= cfg.agentSourceIp || "10.200.4.1";  
    elSubAgentId.value \= cfg.subAgentId \!== undefined ? cfg.subAgentId : 1;  
    elPrimaryCollector.value \= cfg.collectorHost || "sflow-collector-01.corp.internal";  
    elCollectorPort.value \= cfg.port || 6343;  
    elSecondaryCollector.value \= cfg.secondaryCollectorHost || "";  
    elChk64BitCounters.checked \= cfg.counters64Bit \!== false;  
    elChkDropAccounting.checked \= cfg.dropPacketAccounting \!== false;  
    elChkExtendedSwitch.checked \= cfg.extendedSwitchData \!== false;  
    elChkExtendedRouter.checked \= cfg.extendedRouterData \!== false;  
    elCollectorBuffer.value \= cfg.collectorBufferMb || 32;

    updateUI();  
  }

  function updateUI() {  
    const cfg \= getCurrentConfig();

    // 1\. Update Link Sizing & Nyquist Recommendation  
    const rec \= SFlowEngine.calculateRecommendedSamplingRate(cfg.linkSpeedGbps, 400);  
    elCalcSpeedText.textContent \= \`\${cfg.linkSpeedGbps}G\`;  
    elCalcRecRate.textContent \= \`1:\${rec.recommendedRate}\`;

    // 2\. Run Security Audit  
    const audit \= SFlowSecurityAuditor.auditSFlowConfiguration(cfg);  
    elScorecardGrade.textContent \= audit.grade;  
    elScorecardGrade.className \= 'grade-badge ' \+ (audit.grade \=== 'A+' || audit.grade \=== 'A' ? 'grade-a' : (audit.grade \=== 'B' || audit.grade \=== 'C' ? 'grade-b' : 'grade-f'));  
    elScoreFill.style.width \= \`\${audit.percentage}%\`;  
    elScorePoints.textContent \= audit.score;  
    elScorePercentage.textContent \= \`\${audit.percentage}%\`;  
    elPassedCount.textContent \= audit.passedCount;  
    elWarnCount.textContent \= audit.warnCount;  
    elFailCount.textContent \= audit.failCount;

    // Render Checklist  
    elRubricChecklist.innerHTML \= '';  
    audit.checks.forEach(check \=\> {  
      const item \= document.createElement('div');  
      item.className \= 'rubric-item';  
      const statusClass \= check.status \=== 'PASS' ? 'status-pass' : (check.status \=== 'WARN' ? 'status-warn' : 'status-fail');  
      const icon \= check.status \=== 'PASS' ? '✅' : (check.status \=== 'WARN' ? '⚠️' : '❌');  
      item.innerHTML \= \`  
        \<div\>  
          \<span class="rubric-status \${statusClass}"\>\${icon} \${check.status}\</span\>  
          \<strong\>\${check.name}\</strong\> (\${check.score}/\${check.maxScore} pts)  
          \<div class="help-text"\>\${check.details}\</div\>  
        \</div\>  
      \`;  
      elRubricChecklist.appendChild(item);  
    });

    // 3\. Build Binary XDR Wireframe  
    const dgram \= SFlowEngine.buildSFlowDatagram({  
      agentIp: cfg.agentSourceIp,  
      subAgentId: cfg.subAgentId,  
      samplingRate: cfg.samplingRate,  
      headerBytes: cfg.headerBytes,  
      ifSpeedBps: BigInt(cfg.linkSpeedGbps) \* 1000000000n  
    });  
    elTotalPacketSize.textContent \= \`\${dgram.totalLength} bytes\`;  
    elHexDumpCode.textContent \= dgram.hexDump;

    // 4\. Update Synthesizer  
    updateSynthesizerPreview(cfg);  
  }

  function updateSynthesizerPreview(cfg) {  
    let output \= '';  
    if (activeVendorTab \=== 'tab-arista') {  
      output \= SFlowConfigSynthesizer.synthesizeAristaEOS(cfg);  
    } else if (activeVendorTab \=== 'tab-cisco') {  
      output \= SFlowConfigSynthesizer.synthesizeCiscoNXOS(cfg);  
    } else if (activeVendorTab \=== 'tab-juniper') {  
      output \= SFlowConfigSynthesizer.synthesizeJuniperJunos(cfg);  
    } else if (activeVendorTab \=== 'tab-hsflowd') {  
      output \= SFlowConfigSynthesizer.synthesizeHsflowd(cfg);  
    } else if (activeVendorTab \=== 'tab-fastnetmon') {  
      output \= SFlowConfigSynthesizer.synthesizeFastNetMon(cfg);  
    } else if (activeVendorTab \=== 'tab-python') {  
      output \= SFlowConfigSynthesizer.synthesizePythonClient(cfg);  
    }  
    elVendorSynthesizerCode.textContent \= output;  
  }

  function setupEventHandlers() {  
    const inputs \= \[  
      elLinkSpeed, elSamplingRate, elHeaderBytes, elCounterPolling,  
      elAgentIp, elSubAgentId, elPrimaryCollector, elCollectorPort,  
      elSecondaryCollector, elChk64BitCounters, elChkDropAccounting,  
      elChkExtendedSwitch, elChkExtendedRouter, elCollectorBuffer  
    \];  
    inputs.forEach(el \=\> {  
      el.addEventListener('input', updateUI);  
      el.addEventListener('change', updateUI);  
    });

    // Link speed change updates recommended rate  
    elLinkSpeed.addEventListener('change', () \=\> {  
      const speed \= parseFloat(elLinkSpeed.value) || 100;  
      const rec \= SFlowEngine.calculateRecommendedSamplingRate(speed, 400);  
      elSamplingRate.value \= rec.recommendedRate;  
      updateUI();  
    });

    // Preset selector  
    elPresetSelect.addEventListener('change', () \=\> {  
      const presetKey \= elPresetSelect.value;  
      if (PRESETS\[presetKey\]) applyConfig(PRESETS\[presetKey\]);  
    });

    elBtnResetPreset.addEventListener('click', () \=\> {  
      const presetKey \= elPresetSelect.value;  
      if (PRESETS\[presetKey\]) applyConfig(PRESETS\[presetKey\]);  
    });

    // Export JSON  
    elBtnExportJson.addEventListener('click', () \=\> {  
      const cfg \= getCurrentConfig();  
      const blob \= new Blob(\[JSON.stringify(cfg, null, 2)\], { type: 'application/json' });  
      const url \= URL.createObjectURL(blob);  
      const a \= document.createElement('a');  
      a.href \= url;  
      a.download \= \`sflow-config-\${cfg.linkSpeedGbps}g-rate\${cfg.samplingRate}.json\`;  
      a.click();  
      URL.revokeObjectURL(url);  
    });

    // Vendor Tabs  
    const vendorTabBtns \= document.querySelectorAll('\#vendor-tabs .tab-btn');  
    vendorTabBtns.forEach(btn \=\> {  
      btn.addEventListener('click', () \=\> {  
        vendorTabBtns.forEach(b \=\> b.classList.remove('active'));  
        btn.classList.add('active');  
        activeVendorTab \= btn.getAttribute('data-tab');  
        updateSynthesizerPreview(getCurrentConfig());  
      });  
    });

    // Copy buttons  
    elBtnCopyHexDump.addEventListener('click', () \=\> {  
      navigator.clipboard.writeText(elHexDumpCode.textContent).then(() \=\> {  
        const orig \= elBtnCopyHexDump.textContent;  
        elBtnCopyHexDump.textContent \= 'Copied\!';  
        setTimeout(() \=\> elBtnCopyHexDump.textContent \= orig, 1500);  
      });  
    });

    elBtnCopyConfig.addEventListener('click', () \=\> {  
      navigator.clipboard.writeText(elVendorSynthesizerCode.textContent).then(() \=\> {  
        const orig \= elBtnCopyConfig.textContent;  
        elBtnCopyConfig.textContent \= 'Copied\!';  
        setTimeout(() \=\> elBtnCopyConfig.textContent \= orig, 1500);  
      });  
    });

    // Download Config  
    elBtnDownloadConfig.addEventListener('click', () \=\> {  
      const code \= elVendorSynthesizerCode.textContent;  
      let filename \= 'sflow-config.txt';  
      if (activeVendorTab \=== 'tab-arista') filename \= 'arista-eos-sflow.cfg';  
      else if (activeVendorTab \=== 'tab-cisco') filename \= 'cisco-nxos-sflow.cfg';  
      else if (activeVendorTab \=== 'tab-juniper') filename \= 'juniper-sflow.set';  
      else if (activeVendorTab \=== 'tab-hsflowd') filename \= 'hsflowd.conf';  
      else if (activeVendorTab \=== 'tab-fastnetmon') filename \= 'fastnetmon.conf';  
      else if (activeVendorTab \=== 'tab-python') filename \= 'test\_sflow\_sender.py';

      const blob \= new Blob(\[code\], { type: 'text/plain' });  
      const url \= URL.createObjectURL(blob);  
      const a \= document.createElement('a');  
      a.href \= url;  
      a.download \= filename;  
      a.click();  
      URL.revokeObjectURL(url);  
    });  
  }

  // Init  
  setupEventHandlers();  
  applyConfig(PRESETS\['datacenter-spine-400g-arista'\]);  
})();  
