/\*\*  
 \* sFlowForge Pro \- sFlow v5 (RFC 3176 / InMon) XDR Telemetry & Binary Framing Engine  
 \* Generates and parses sFlow v5 Datagrams, Flow Samples, and Interface Counter Samples.  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.SFlowEngine \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  // sFlow Sample & Record Tags  
  const SFLOW\_TAGS \= {  
    FLOW\_SAMPLE: 1,  
    COUNTER\_SAMPLE: 2,  
    EXPANDED\_FLOW\_SAMPLE: 3,  
    EXPANDED\_COUNTER\_SAMPLE: 4,  
    // Flow Records  
    RECORD\_RAW\_PACKET: 1,  
    RECORD\_EXTENDED\_SWITCH: 1001,  
    RECORD\_EXTENDED\_ROUTER: 1002,  
    // Counter Records  
    RECORD\_GENERIC\_INTERFACE: 1,  
    RECORD\_ETHERNET\_INTERFACE: 2  
  };

  /\*\*  
   \* Recommends optimal sFlow sampling rate based on port speed and target samples/sec  
   \*/  
  function calculateRecommendedSamplingRate(linkSpeedGbps, targetSamplesPerSec) {  
    const speed \= parseFloat(linkSpeedGbps) || 10;  
    const target \= parseFloat(targetSamplesPerSec) || 400; // standard 400 samples/sec target

    // Typical packet rates assuming mixed traffic average packet size 800 bytes (1 packet \= 6400 bits)  
    // pps \= (speed \* 10^9) / 6400  
    const estPps \= (speed \* 1000000000\) / 6400;  
    const rawRate \= Math.round(estPps / target);

    // Round to nearest clean standard rate  
    let recommended \= 1000;  
    if (rawRate \<= 500\) recommended \= 500;  
    else if (rawRate \<= 1000\) recommended \= 1000;  
    else if (rawRate \<= 2000\) recommended \= 2000;  
    else if (rawRate \<= 5000\) recommended \= 5000;  
    else if (rawRate \<= 10000\) recommended \= 10000;  
    else if (rawRate \<= 20000\) recommended \= 20000;  
    else recommended \= 40000;

    return {  
      linkSpeedGbps: speed,  
      estimatedPps: Math.round(estPps),  
      targetSamplesPerSec: target,  
      recommendedRate: recommended  
    };  
  }

  /\*\*  
   \* Encodes an IP address string to 4 hex bytes  
   \*/  
  function ipToHex(ipStr) {  
    const parts \= (ipStr || '10.200.4.1').split('.').map(n \=\> parseInt(n, 10\) || 0);  
    return parts.map(b \=\> b.toString(16).padStart(2, '0')).join(' ');  
  }

  /\*\*  
   \* Builds an RFC 3176 / InMon sFlow v5 Datagram Wireframe & Hex Dump  
   \*/  
  function buildSFlowDatagram(options) {  
    const opts \= options || {};  
    const version \= 5;  
    const agentIp \= opts.agentIp || '10.200.4.1';  
    const subAgentId \= parseInt(opts.subAgentId \!== undefined ? opts.subAgentId : 1, 10);  
    const datagramSeqNumber \= parseInt(opts.datagramSeqNumber || 1001, 10);  
    const uptimeMs \= parseInt(opts.uptimeMs || 8492010, 10);

    // Flow Sample Settings  
    const flowSeqNumber \= parseInt(opts.flowSeqNumber || 2501, 10);  
    const sourceId \= parseInt(opts.sourceId || 2, 10); // ifIndex  
    const samplingRate \= parseInt(opts.samplingRate || 1000, 10);  
    const samplePool \= parseInt(opts.samplePool || (flowSeqNumber \* samplingRate), 10);  
    const droppedPackets \= parseInt(opts.droppedPackets || 0, 10);  
    const inputInterface \= parseInt(opts.inputInterface || 2, 10);  
    const outputInterface \= parseInt(opts.outputInterface || 4, 10);

    // Raw Packet Header Data  
    const headerProtocol \= 1; // 1 \= ETHERNET-ISO88023  
    const frameLength \= parseInt(opts.frameLength || 1514, 10);  
    const strippedOctets \= parseInt(opts.strippedOctets || 4, 10); // FCS 4 bytes  
    const headerBytes \= parseInt(opts.headerBytes || 128, 10);

    // Extended Switch Data  
    const srcVlan \= parseInt(opts.srcVlan || 100, 10);  
    const srcPriority \= parseInt(opts.srcPriority || 0, 10);  
    const dstVlan \= parseInt(opts.dstVlan || 200, 10);  
    const dstPriority \= parseInt(opts.dstPriority || 0, 10);

    // Counter Sample Settings  
    const counterSeqNumber \= parseInt(opts.counterSeqNumber || 42, 10);  
    const ifSpeedBps \= BigInt(opts.ifSpeedBps || 100000000000); // 100 Gbps  
    const ifInOctets \= BigInt(opts.ifInOctets || 4999942144);  
    const ifInPackets \= parseInt(opts.ifInPackets || 3448850, 10);  
    const ifOutOctets \= BigInt(opts.ifOutOctets || 3829104820);  
    const ifOutPackets \= parseInt(opts.ifOutPackets || 2981042, 10);

    const hexParts \= \[\];

    // \--- 1\. SFLOW DATAGRAM HEADER (28 bytes) \---  
    // sflow\_version (4 bytes) \= 5  
    hexParts.push('00 00 00 05');  
    // address\_type (4 bytes): 1 \= IPv4, 2 \= IPv6  
    hexParts.push('00 00 00 01');  
    // agent\_address (4 bytes)  
    hexParts.push(ipToHex(agentIp));  
    // sub\_agent\_id (4 bytes)  
    hexParts.push(subAgentId.toString(16).padStart(8, '0'));  
    // sequence\_number (4 bytes)  
    hexParts.push(datagramSeqNumber.toString(16).padStart(8, '0'));  
    // uptime (4 bytes)  
    hexParts.push(uptimeMs.toString(16).padStart(8, '0'));  
    // num\_samples (4 bytes) \= 2 (1 Flow Sample \+ 1 Counter Sample)  
    hexParts.push('00 00 00 02');

    // \--- 2\. SAMPLE 1: FLOW\_SAMPLE (Enterprise 0, Format 1\) \---  
    const sample1Tag \= SFLOW\_TAGS.FLOW\_SAMPLE; // 1  
    // Flow Sample Header: Seq(4) \+ SrcId(4) \+ Rate(4) \+ Pool(4) \+ Drops(4) \+ InIf(4) \+ OutIf(4) \+ NumRecords(4) \= 32 bytes  
    // Record 1: RAW\_PACKET\_HEADER (Format 1\)  
    // Tag(4) \+ Len(4) \+ Protocol(4) \+ FrameLen(4) \+ Stripped(4) \+ HeaderLen(4) \+ PayloadBytes(headerBytes) \+ XDR Padding  
    const rawPad \= (4 \- (headerBytes % 4)) % 4;  
    const rawRecordPayloadLen \= 16 \+ headerBytes \+ rawPad;  
    const rawRecordTotalLen \= 8 \+ rawRecordPayloadLen;

    // Record 2: EXTENDED\_SWITCH\_DATA (Format 1001\)  
    // Tag(4) \+ Len(4) \+ src\_vlan(4) \+ src\_prio(4) \+ dst\_vlan(4) \+ dst\_prio(4) \= 24 bytes  
    const extSwitchRecordLen \= 24;

    const sample1PayloadLen \= 32 \+ rawRecordTotalLen \+ extSwitchRecordLen;

    // Sample Tag (Enterprise=0, Format=1 \=\> (0 \<\< 12\) \+ 1 \= 1\)  
    hexParts.push('00 00 00 01');  
    // Sample Length  
    hexParts.push(sample1PayloadLen.toString(16).padStart(8, '0'));  
    // Flow Sample Fields  
    hexParts.push(flowSeqNumber.toString(16).padStart(8, '0'));  
    hexParts.push(((0 \<\< 24\) | sourceId).toString(16).padStart(8, '0')); // 0 \= ifIndex  
    hexParts.push(samplingRate.toString(16).padStart(8, '0'));  
    hexParts.push(samplePool.toString(16).padStart(8, '0'));  
    hexParts.push(droppedPackets.toString(16).padStart(8, '0'));  
    hexParts.push(inputInterface.toString(16).padStart(8, '0'));  
    hexParts.push(outputInterface.toString(16).padStart(8, '0'));  
    // num\_flow\_records \= 2  
    hexParts.push('00 00 00 02');

    // Flow Record 1: RAW\_PACKET\_HEADER  
    hexParts.push('00 00 00 01'); // Record Tag 1  
    hexParts.push(rawRecordPayloadLen.toString(16).padStart(8, '0')); // Record Length  
    hexParts.push('00 00 00 01'); // header\_protocol: 1 (Ethernet)  
    hexParts.push(frameLength.toString(16).padStart(8, '0'));  
    hexParts.push(strippedOctets.toString(16).padStart(8, '0'));  
    hexParts.push(headerBytes.toString(16).padStart(8, '0'));

    // Simulated Raw Ethernet Frame Header bytes (up to headerBytes)  
    // Ethernet: Dst MAC (6B), Src MAC (6B), 802.1Q (4B), IPv4 Header (20B), TCP Header (20B)  
    const ethHeaderHex \= '00 1c 73 00 00 01 00 1c 73 00 00 02 81 00 00 64 08 00 45 00 00 3c 1a 2b 40 00 40 06 b2 c1 0a c8 04 01 c6 33 64 2c d0 00 01 bb 00 00 03 e9 00 00 00 00 a0 02 72 10 3a 4f 00 00 02 04 05 b4';  
    const ethBytes \= ethHeaderHex.split(' ');  
    for (let i \= 0; i \< headerBytes; i++) {  
      hexParts.push(ethBytes\[i % ethBytes.length\]);  
    }  
    if (rawPad \> 0\) {  
      hexParts.push('00 '.repeat(rawPad).trim());  
    }

    // Flow Record 2: EXTENDED\_SWITCH\_DATA  
    hexParts.push('00 00 03 e9'); // Tag 1001 (0x03e9)  
    hexParts.push('00 00 00 10'); // Length 16 bytes  
    hexParts.push(srcVlan.toString(16).padStart(8, '0'));  
    hexParts.push(srcPriority.toString(16).padStart(8, '0'));  
    hexParts.push(dstVlan.toString(16).padStart(8, '0'));  
    hexParts.push(dstPriority.toString(16).padStart(8, '0'));

    // \--- 3\. SAMPLE 2: COUNTER\_SAMPLE (Enterprise 0, Format 2\) \---  
    // Counter Sample Header: Seq(4) \+ SrcId(4) \+ NumRecords(4) \= 12 bytes  
    // Record 1: GENERIC\_INTERFACE\_COUNTERS (Format 1\)  
    // Tag(4) \+ Len(4) \+ ifIndex(4) \+ ifType(4) \+ ifSpeed(8) \+ ifDirection(4) \+ ifStatus(4) \+ inOctets(8) \+ inPkts(4) \+ inMulti(4) \+ inBcast(4) \+ inDisc(4) \+ inErr(4) \+ inUnk(4) \+ outOctets(8) \+ outPkts(4) \+ outMulti(4) \+ outBcast(4) \+ outDisc(4) \+ outErr(4) \+ promisc(4) \= 88 bytes  
    const genericCounterRecordLen \= 88;  
    const sample2PayloadLen \= 12 \+ genericCounterRecordLen;

    hexParts.push('00 00 00 02'); // Sample Tag 2  
    hexParts.push(sample2PayloadLen.toString(16).padStart(8, '0'));  
    hexParts.push(counterSeqNumber.toString(16).padStart(8, '0'));  
    hexParts.push(((0 \<\< 24\) | sourceId).toString(16).padStart(8, '0'));  
    hexParts.push('00 00 00 01'); // 1 counter record

    // Generic Interface Record  
    hexParts.push('00 00 00 01'); // Tag 1  
    hexParts.push('00 00 00 50'); // Length 80 bytes (0x50)  
    hexParts.push(sourceId.toString(16).padStart(8, '0')); // ifIndex  
    hexParts.push('00 00 00 06'); // ifType 6 (ethernetCsmacd)  
    hexParts.push(ifSpeedBps.toString(16).padStart(16, '0')); // 64-bit ifSpeed  
    hexParts.push('00 00 00 01'); // ifDirection 1 (full-duplex)  
    hexParts.push('00 00 00 03'); // ifStatus 3 (admin up, oper up)  
    hexParts.push(ifInOctets.toString(16).padStart(16, '0')); // 64-bit inOctets  
    hexParts.push(ifInPackets.toString(16).padStart(8, '0'));  
    hexParts.push('00 00 01 20'); // inMulti  
    hexParts.push('00 00 00 40'); // inBcast  
    hexParts.push('00 00 00 00'); // inDiscards  
    hexParts.push('00 00 00 00'); // inErrors  
    hexParts.push('00 00 00 00'); // inUnknownProtos  
    hexParts.push(ifOutOctets.toString(16).padStart(16, '0')); // 64-bit outOctets  
    hexParts.push(ifOutPackets.toString(16).padStart(8, '0'));  
    hexParts.push('00 00 00 f0'); // outMulti  
    hexParts.push('00 00 00 20'); // outBcast  
    hexParts.push('00 00 00 00'); // outDiscards  
    hexParts.push('00 00 00 00'); // outErrors  
    hexParts.push('00 00 00 00'); // promisc

    const rawHex \= hexParts.join(' ').replace(/\\s+/g, ' ');  
    const totalBytes \= rawHex.split(' ').length;

    return {  
      version,  
      agentIp,  
      subAgentId,  
      datagramSeqNumber,  
      uptimeMs,  
      numSamples: 2,  
      totalLength: totalBytes,  
      flowSample: {  
        flowSeqNumber,  
        sourceId,  
        samplingRate,  
        samplePool,  
        droppedPackets,  
        inputInterface,  
        outputInterface,  
        rawPacketHeader: {  
          headerProtocol: 'ETHERNET-ISO88023',  
          frameLength,  
          strippedOctets,  
          headerBytes  
        },  
        extendedSwitch: {  
          srcVlan,  
          srcPriority,  
          dstVlan,  
          dstPriority  
        }  
      },  
      counterSample: {  
        counterSeqNumber,  
        sourceId,  
        ifSpeedGbps: Number(ifSpeedBps / 1000000000n),  
        ifInOctets: ifInOctets.toString(),  
        ifOutOctets: ifOutOctets.toString(),  
        ifInPackets,  
        ifOutPackets  
      },  
      rawHex,  
      hexDump: formatHexDump(rawHex)  
    };  
  }

  function formatHexDump(rawHex) {  
    const bytes \= rawHex.split(' ');  
    const lines \= \[\];  
    for (let i \= 0; i \< bytes.length; i \+= 16\) {  
      const chunk \= bytes.slice(i, i \+ 16);  
      const offset \= i.toString(16).padStart(4, '0');  
      const hexLine \= chunk.join(' ').padEnd(48, ' ');  
      const ascii \= chunk.map(b \=\> {  
        const val \= parseInt(b, 16);  
        return (val \>= 32 && val \<= 126\) ? String.fromCharCode(val) : '.';  
      }).join('');  
      lines.push(\`\${offset}:  \${hexLine}  |\${ascii}|\`);  
    }  
    return lines.join('\\n');  
  }

  return {  
    SFLOW\_TAGS,  
    calculateRecommendedSamplingRate,  
    buildSFlowDatagram,  
    formatHexDump  
  };  
}));  
