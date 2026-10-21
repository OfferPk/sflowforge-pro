/\*\*  
 \* sFlowForge Pro \- 10-Point Zero-Trust Sampling & Observability Hygiene Auditor  
 \* Evaluates sFlow v5 exporter configurations, sampling rates, header capture depths, and collector buffers (0-100%, Grades A+ to F).  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.SFlowSecurityAuditor \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  function auditSFlowConfiguration(config) {  
    const cfg \= config || {};  
    const checks \= \[\];  
    let totalScore \= 0;  
    const maxScore \= 100;

    // 1\. Header Truncation Depth (15 pts)  
    const headerBytes \= parseInt(cfg.headerBytes || 128, 10);  
    if (headerBytes \>= 128\) {  
      checks.push({  
        id: 'header\_depth',  
        name: 'Header Truncation Depth (\>= 128 Bytes)',  
        score: 15,  
        maxScore: 15,  
        status: 'PASS',  
        details: \`Capture depth of \${headerBytes} bytes captures complete Ethernet, 802.1Q VLAN, IPv4/IPv6, TCP/UDP, and encapsulated VXLAN/Geneve tunnel headers.\`  
      });  
      totalScore \+= 15;  
    } else if (headerBytes \>= 96\) {  
      checks.push({  
        id: 'header\_depth',  
        name: 'Header Truncation Depth (\>= 128 Bytes)',  
        score: 8,  
        maxScore: 15,  
        status: 'WARN',  
        details: \`Capture depth of \${headerBytes} bytes handles standard IP packets, but truncates VXLAN, Geneve, and GRE encapsulated inner headers.\`  
      });  
      totalScore \+= 8;  
    } else {  
      checks.push({  
        id: 'header\_depth',  
        name: 'Header Truncation Depth (\>= 128 Bytes)',  
        score: 0,  
        maxScore: 15,  
        status: 'FAIL',  
        details: \`Insufficient header depth (\${headerBytes} bytes). Truncates TCP options and payload markers, breaking deep flow classification.\`  
      });  
    }

    // 2\. Sampling Rate Sizing vs Link Speed (15 pts)  
    const speedGbps \= parseFloat(cfg.linkSpeedGbps || 100);  
    const rate \= parseInt(cfg.samplingRate || 1000, 10);  
    let ratePass \= false;  
    let rateDetails \= '';

    if (speedGbps \>= 400\) {  
      ratePass \= rate \<= 8000 && rate \>= 2000;  
      rateDetails \= \`400G link with 1:\${rate} sampling. Balanced between microburst fidelity and control-plane CPU safety.\`;  
    } else if (speedGbps \>= 100\) {  
      ratePass \= rate \<= 4000 && rate \>= 1000;  
      rateDetails \= \`100G link with 1:\${rate} sampling. Optimal statistical coverage for backbone observability.\`;  
    } else if (speedGbps \>= 10\) {  
      ratePass \= rate \<= 2000 && rate \>= 500;  
      rateDetails \= \`10G/25G link with 1:\${rate} sampling. High-fidelity sampling without ASIC queue load.\`;  
    } else {  
      ratePass \= rate \<= 1000 && rate \>= 100;  
      rateDetails \= \`1G link with 1:\${rate} sampling. Accurate campus edge visibility.\`;  
    }

    if (ratePass) {  
      checks.push({  
        id: 'sampling\_calibration',  
        name: 'Nyquist-Calibrated Sampling Rate Sizing',  
        score: 15,  
        maxScore: 15,  
        status: 'PASS',  
        details: rateDetails  
      });  
      totalScore \+= 15;  
    } else if (rate \> 20000\) {  
      checks.push({  
        id: 'sampling\_calibration',  
        name: 'Nyquist-Calibrated Sampling Rate Sizing',  
        score: 2,  
        maxScore: 15,  
        status: 'FAIL',  
        details: \`Excessive sampling rate (1:\${rate}) causes severe statistical aliasing; misses transient DDoS microbursts and port scans.\`  
      });  
      totalScore \+= 2;  
    } else {  
      checks.push({  
        id: 'sampling\_calibration',  
        name: 'Nyquist-Calibrated Sampling Rate Sizing',  
        score: 7,  
        maxScore: 15,  
        status: 'WARN',  
        details: \`Sampling rate of 1:\${rate} on \${speedGbps}G link is non-optimal. Monitor switch CPU and collector socket load.\`  
      });  
      totalScore \+= 7;  
    }

    // 3\. Periodic Counter Polling Cadence (10 pts)  
    const pollingSec \= parseInt(cfg.pollingIntervalSeconds || 20, 10);  
    if (pollingSec \<= 20 && pollingSec \>= 5\) {  
      checks.push({  
        id: 'counter\_polling',  
        name: 'Periodic Interface Counter Polling (\<= 20s)',  
        score: 10,  
        maxScore: 10,  
        status: 'PASS',  
        details: \`Frequent interface counter polling (\${pollingSec}s). Delivers near real-time SNMP-grade interface octet & error statistics.\`  
      });  
      totalScore \+= 10;  
    } else if (pollingSec \<= 60\) {  
      checks.push({  
        id: 'counter\_polling',  
        name: 'Periodic Interface Counter Polling (\<= 20s)',  
        score: 5,  
        maxScore: 10,  
        status: 'WARN',  
        details: \`Counter polling interval of \${pollingSec}s is moderate. Smooths out sub-minute interface utilization bursts.\`  
      });  
      totalScore \+= 5;  
    } else {  
      checks.push({  
        id: 'counter\_polling',  
        name: 'Periodic Interface Counter Polling (\<= 20s)',  
        score: 2,  
        maxScore: 10,  
        status: 'FAIL',  
        details: \`Infrequent polling interval (\${pollingSec}s). Blind to transient link saturation and queue drops.\`  
      });  
      totalScore \+= 2;  
    }

    // 4\. 64-bit High-Capacity Interface Counters (10 pts)  
    const is64BitCounters \= cfg.counters64Bit \!== false;  
    if (is64BitCounters) {  
      checks.push({  
        id: 'counters\_64bit',  
        name: '64-Bit High-Capacity Interface Counters',  
        score: 10,  
        maxScore: 10,  
        status: 'PASS',  
        details: '64-bit ifInOctets & ifOutOctets active. Prevents integer rollover under 100G/400G line-rate utilization.'  
      });  
      totalScore \+= 10;  
    } else {  
      checks.push({  
        id: 'counters\_64bit',  
        name: '64-Bit High-Capacity Interface Counters',  
        score: 0,  
        maxScore: 10,  
        status: 'FAIL',  
        details: '32-bit counters detected. Rolls over in \<35 seconds on 1G links and sub-second on 100G, corrupting metrics.'  
      });  
    }

    // 5\. Collector Redundancy & High Availability (10 pts)  
    const hasSecCollector \= \!\!cfg.secondaryCollectorHost;  
    if (hasSecCollector) {  
      checks.push({  
        id: 'collector\_redundancy',  
        name: 'Dual-Collector High Availability Export',  
        score: 10,  
        maxScore: 10,  
        status: 'PASS',  
        details: \`Redundant secondary collector configured (\${cfg.secondaryCollectorHost}). Eliminates single point of telemetry failure.\`  
      });  
      totalScore \+= 10;  
    } else {  
      checks.push({  
        id: 'collector\_redundancy',  
        name: 'Dual-Collector High Availability Export',  
        score: 2,  
        maxScore: 10,  
        status: 'WARN',  
        details: 'Single collector target configured. Receiver host reboot or maintenance causes total flow blindness.'  
      });  
      totalScore \+= 2;  
    }

    // 6\. Dropped Packet Accounting & Buffer Health (10 pts)  
    const dropAccounting \= cfg.dropPacketAccounting \!== false;  
    if (dropAccounting) {  
      checks.push({  
        id: 'drop\_accounting',  
        name: 'ASIC Queue Drop Packet Telemetry Accounting',  
        score: 10,  
        maxScore: 10,  
        status: 'PASS',  
        details: 'dropped\_packets sample field enabled. Explicitly alerts SIEM when switch ASIC sampling queues suffer overflow.'  
      });  
      totalScore \+= 10;  
    } else {  
      checks.push({  
        id: 'drop\_accounting',  
        name: 'ASIC Queue Drop Packet Telemetry Accounting',  
        score: 0,  
        maxScore: 10,  
        status: 'FAIL',  
        details: 'Drop packet accounting disabled. Collector cannot distinguish between zero traffic and ASIC sample queue drops.'  
      });  
    }

    // 7\. Agent IP Pinning & Sub-Agent Scoping (10 pts)  
    const hasAgentIp \= \!\!cfg.agentSourceIp;  
    const subAgentId \= parseInt(cfg.subAgentId \!== undefined ? cfg.subAgentId : 1, 10);  
    if (hasAgentIp && subAgentId \> 0\) {  
      checks.push({  
        id: 'agent\_scoping',  
        name: 'Dedicated Agent IP & Sub-Agent ID Scoping',  
        score: 10,  
        maxScore: 10,  
        status: 'PASS',  
        details: \`Stable Loopback Agent IP (\${cfg.agentSourceIp}) and sub-agent ID (\${subAgentId}) pinned. Prevents interface IP flapping.\`  
      });  
      totalScore \+= 10;  
    } else if (hasAgentIp) {  
      checks.push({  
        id: 'agent\_scoping',  
        name: 'Dedicated Agent IP & Sub-Agent ID Scoping',  
        score: 5,  
        maxScore: 10,  
        status: 'WARN',  
        details: 'Agent IP configured, but sub-agent ID is default 0\. Can cause collision in modular chassis systems.'  
      });  
      totalScore \+= 5;  
    } else {  
      checks.push({  
        id: 'agent\_scoping',  
        name: 'Dedicated Agent IP & Sub-Agent ID Scoping',  
        score: 0,  
        maxScore: 10,  
        status: 'FAIL',  
        details: 'No static agent IP configured. Switches may export from random egress physical interfaces.'  
      });  
    }

    // 8\. Extended Switch & Router Data Attachment (10 pts)  
    const extSwitch \= cfg.extendedSwitchData \!== false;  
    const extRouter \= cfg.extendedRouterData \!== false;  
    if (extSwitch && extRouter) {  
      checks.push({  
        id: 'extended\_data',  
        name: 'Extended Switch (VLAN) & Router (Next-Hop) Records',  
        score: 10,  
        maxScore: 10,  
        status: 'PASS',  
        details: 'Both Extended Switch Data (VLANs, 802.1p) and Extended Router Data (Next-hop IP, subnets) attached to samples.'  
      });  
      totalScore \+= 10;  
    } else if (extSwitch || extRouter) {  
      checks.push({  
        id: 'extended\_data',  
        name: 'Extended Switch (VLAN) & Router (Next-Hop) Records',  
        score: 5,  
        maxScore: 10,  
        status: 'WARN',  
        details: 'Partial extended records attached. Missing VLAN priority or routing next-hop context.'  
      });  
      totalScore \+= 5;  
    } else {  
      checks.push({  
        id: 'extended\_data',  
        name: 'Extended Switch (VLAN) & Router (Next-Hop) Records',  
        score: 0,  
        maxScore: 10,  
        status: 'FAIL',  
        details: 'Zero extended records attached. Samples contain only raw L2 packet bytes without switch topology metadata.'  
      });  
    }

    // 9\. Collector Socket Buffer Sizing (SO\_RCVBUF \>= 16MB) (5 pts)  
    const socketBufferMb \= parseInt(cfg.collectorBufferMb || 16, 10);  
    if (socketBufferMb \>= 16\) {  
      checks.push({  
        id: 'socket\_buffer',  
        name: 'Collector Kernel Socket Buffer Hardening (\>= 16MB)',  
        score: 5,  
        maxScore: 5,  
        status: 'PASS',  
        details: \`Collector receive buffer tuned to \${socketBufferMb}MB. Prevents kernel UDP socket drops during traffic microbursts.\`  
      });  
      totalScore \+= 5;  
    } else {  
      checks.push({  
        id: 'socket\_buffer',  
        name: 'Collector Kernel Socket Buffer Hardening (\>= 16MB)',  
        score: 0,  
        maxScore: 5,  
        status: 'WARN',  
        details: \`Default small UDP buffer (\${socketBufferMb}MB). High risk of silent kernel UDP packet drops under burst DDoS.\`  
      });  
    }

    // 10\. Flow Directional Scoping (5 pts)  
    const isBidirectional \= cfg.direction \=== 'both' || (cfg.ingressEnabled \!== false && cfg.egressEnabled \!== false);  
    if (isBidirectional) {  
      checks.push({  
        id: 'directional\_scoping',  
        name: 'Bidirectional (Ingress \+ Egress) Flow Sampling',  
        score: 5,  
        maxScore: 5,  
        status: 'PASS',  
        details: 'Both ingress and egress packet sampling active across monitored switchports.'  
      });  
      totalScore \+= 5;  
    } else {  
      checks.push({  
        id: 'directional\_scoping',  
        name: 'Bidirectional (Ingress \+ Egress) Flow Sampling',  
        score: 2,  
        maxScore: 5,  
        status: 'WARN',  
        details: 'Unidirectional sampling only. Egress dropped packets or traffic modifications are invisible.'  
      });  
      totalScore \+= 2;  
    }

    // Compute Overall Grade  
    let grade \= 'F';  
    if (totalScore \>= 95\) grade \= 'A+';  
    else if (totalScore \>= 90\) grade \= 'A';  
    else if (totalScore \>= 80\) grade \= 'B';  
    else if (totalScore \>= 70\) grade \= 'C';  
    else if (totalScore \>= 60\) grade \= 'D';

    return {  
      score: totalScore,  
      maxScore: maxScore,  
      percentage: Math.round((totalScore / maxScore) \* 100),  
      grade: grade,  
      checks: checks,  
      passedCount: checks.filter(c \=\> c.status \=== 'PASS').length,  
      warnCount: checks.filter(c \=\> c.status \=== 'WARN').length,  
      failCount: checks.filter(c \=\> c.status \=== 'FAIL').length  
    };  
  }

  return {  
    auditSFlowConfiguration  
  };  
}));  
