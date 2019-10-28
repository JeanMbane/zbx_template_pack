
# Template Server IBM IMM SNMPv2

## Overview

For Zabbix version: 3.2  
for IMM2 and IMM1 IBM serverX hardware

This template was tested on:

- IBM System x3550 M2 with IMM1
- IBM x3250M3 with IMM1
- IBM x3550M5 with IMM2
- System x3550 M3 with IMM1

## Setup

Refer to the vendor documentation.

## Zabbix configuration

No specific Zabbix configuration is required.

### Macros used

|Name|Description|Default|
|----|-----------|-------|
|{$DISK_OK_STATUS}|<p>-</p>|`Normal`|
|{$FAN_OK_STATUS}|<p>-</p>|`Normal`|
|{$HEALTH_CRIT_STATUS}|<p>-</p>|`2`|
|{$HEALTH_DISASTER_STATUS}|<p>-</p>|`0`|
|{$HEALTH_WARN_STATUS}|<p>-</p>|`4`|
|{$PSU_OK_STATUS}|<p>-</p>|`Normal`|
|{$TEMP_CRIT:"Ambient"}|<p>-</p>|`35`|
|{$TEMP_CRIT_LOW}|<p>-</p>|`5`|
|{$TEMP_CRIT}|<p>-</p>|`60`|
|{$TEMP_WARN:"Ambient"}|<p>-</p>|`30`|
|{$TEMP_WARN}|<p>-</p>|`50`|

## Template links

|Name|
|----|
|Template Module Generic SNMPv2|

## Discovery rules

|Name|Description|Type|Key and additional info|
|----|-----------|----|----|
|Temperature Discovery|<p>Scanning IMM-MIB::tempTable</p>|SNMP|tempDescr.discovery<p>**Filter**:</p>AND_OR <p>- B: {#SNMPVALUE} MATCHES_REGEX `(DIMM|PSU|PCH|RAID|RR|PCI).*`</p>|
|Temperature Discovery Ambient|<p>Scanning IMM-MIB::tempTable with Ambient filter</p>|SNMP|tempDescr.discovery.ambient<p>**Filter**:</p>AND_OR <p>- B: {#SNMPVALUE} MATCHES_REGEX `Ambient.*`</p>|
|Temperature Discovery CPU|<p>Scanning IMM-MIB::tempTable with CPU filter</p>|SNMP|tempDescr.discovery.cpu<p>**Filter**:</p>AND_OR <p>- B: {#SNMPVALUE} MATCHES_REGEX `CPU [0-9]* Temp`</p>|
|PSU Discovery|<p>IMM-MIB::powerFruName</p>|SNMP|psu.discovery|
|FAN Discovery|<p>IMM-MIB::fanDescr</p>|SNMP|fan.discovery|
|Physical Disk Discovery|<p>-</p>|SNMP|physicalDisk.discovery|

## Items collected

|Group|Name|Description|Type|Key and additional info|
|-----|----|-----------|----|---------------------|
|Fans|{#FAN_DESCR}: Статус вентилятора|<p>MIB: IMM-MIB</p><p>A description of the fan component status.</p>|SNMP|sensor.fan.status[fanHealthStatus.{#SNMPINDEX}]|
|Inventory|Модель|<p>MIB: IMM-MIB</p>|SNMP|system.hw.model<p>**Preprocessing**:</p><p>- DISCARD_UNCHANGED_HEARTBEAT: `1d`</p>|
|Inventory|Серийный номер|<p>MIB: IMM-MIB</p><p>Machine serial number VPD information</p>|SNMP|system.hw.serialnumber<p>**Preprocessing**:</p><p>- DISCARD_UNCHANGED_HEARTBEAT: `1d`</p>|
|Physical_disks|{#SNMPINDEX}: Статус физического диска|<p>MIB: IMM-MIB</p>|SNMP|system.hw.physicaldisk.status[diskHealthStatus.{#SNMPINDEX}]|
|Physical_disks|{#SNMPINDEX}: Код производителя диска|<p>MIB: IMM-MIB</p><p>disk module FRU name.</p>|SNMP|system.hw.physicaldisk.part_number[diskFruName.{#SNMPINDEX}]|
|Power_supply|{#PSU_DESCR}: Статус блока питания|<p>MIB: IMM-MIB</p><p>A description of the power module status.</p>|SNMP|sensor.psu.status[powerHealthStatus.{#SNMPINDEX}]|
|Status|Общий статус системы|<p>MIB: IMM-MIB</p><p>Indicates status of system health for the system in which the IMM resides. Value of 'nonRecoverable' indicates a severe error has occurred and the system may not be functioning. A value of 'critical' indicates that a error has occurred but the system is currently functioning properly. A value of 'nonCritical' indicates that a condition has occurred that may change the state of the system in the future but currently the system is working properly. A value of 'normal' indicates that the system is operating normally.</p>|SNMP|system.status[systemHealthStat.0]|
|Temperature|{#SNMPVALUE}: Температура|<p>MIB: IMM-MIB</p><p>Temperature readings of testpoint: {#SNMPVALUE}</p>|SNMP|sensor.temp.value[tempReading.{#SNMPINDEX}]|
|Temperature|Ambient: Температура|<p>MIB: IMM-MIB</p><p>Temperature readings of testpoint: Ambient</p>|SNMP|sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}]|
|Temperature|CPU: Температура|<p>MIB: IMM-MIB</p><p>Temperature readings of testpoint: CPU</p>|SNMP|sensor.temp.value[tempReading.CPU.{#SNMPINDEX}]|

## Triggers

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----|----|----|
|{#FAN_DESCR}: Статус вентилятора: не норма|<p>Проверьте вентилятор</p>|`{TEMPLATE_NAME:sensor.fan.status[fanHealthStatus.{#SNMPINDEX}].count(#1,{$FAN_OK_STATUS},ne)}=1`|INFO||
|Возможно замена устройства (получен новый серийный номер)|<p>Изменился серийный номер устройства. Подтвердите и закройте.</p>|`{TEMPLATE_NAME:system.hw.serialnumber.diff()}=1 and {TEMPLATE_NAME:system.hw.serialnumber.strlen()}>0`|INFO|<p>Manual close: YES</p>|
|{#SNMPINDEX}: Статус физического диска не норма|<p>Проверьте диск</p>|`{TEMPLATE_NAME:system.hw.physicaldisk.status[diskHealthStatus.{#SNMPINDEX}].count(#1,{$DISK_OK_STATUS},ne)}=1`|WARNING||
|{#PSU_DESCR}: Статус блока питания: не норма|<p>Проверьте блок питания</p>|`{TEMPLATE_NAME:sensor.psu.status[powerHealthStatus.{#SNMPINDEX}].count(#1,{$PSU_OK_STATUS},ne)}=1`|INFO||
|Статус системы: сбой|<p>Проверьте устройство</p>|`{TEMPLATE_NAME:system.status[systemHealthStat.0].count(#1,{$HEALTH_DISASTER_STATUS},eq)}=1`|HIGH||
|Статус системы: авария|<p>Проверьте устройство</p>|`{TEMPLATE_NAME:system.status[systemHealthStat.0].count(#1,{$HEALTH_CRIT_STATUS},eq)}=1`|HIGH|<p>**Depends on**:</p><p>- Статус системы: сбой</p>|
|Статус системы: предупреждение|<p>Проверьте устройство</p>|`{TEMPLATE_NAME:system.status[systemHealthStat.0].count(#1,{$HEALTH_WARN_STATUS},eq)}=1`|WARNING|<p>**Depends on**:</p><p>- Статус системы: авария</p><p>- Статус системы: сбой</p>|
|{#SNMPVALUE}: Температура выше нормы: >{$TEMP_WARN:""}|<p>This trigger uses temperature sensor values as well as temperature sensor status if available</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.{#SNMPINDEX}].avg(5m)}>{$TEMP_WARN:""}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.{#SNMPINDEX}].max(5m)}<{$TEMP_WARN:""}-3`|WARNING|<p>**Depends on**:</p><p>- {#SNMPVALUE}: Температура очень высокая: >{$TEMP_CRIT:""}</p>|
|{#SNMPVALUE}: Температура очень высокая: >{$TEMP_CRIT:""}|<p>This trigger uses temperature sensor values as well as temperature sensor status if available</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.{#SNMPINDEX}].avg(5m)}>{$TEMP_CRIT:""}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.{#SNMPINDEX}].max(5m)}<{$TEMP_CRIT:""}-3`|HIGH||
|{#SNMPVALUE}: Температура слишком низкая: <{$TEMP_CRIT_LOW:""}|<p>-</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.{#SNMPINDEX}].avg(5m)}<{$TEMP_CRIT_LOW:""}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.{#SNMPINDEX}].min(5m)}>{$TEMP_CRIT_LOW:""}+3`|AVERAGE||
|Ambient: Температура выше нормы: >{$TEMP_WARN:"Ambient"}|<p>This trigger uses temperature sensor values as well as temperature sensor status if available</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}].avg(5m)}>{$TEMP_WARN:"Ambient"}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}].max(5m)}<{$TEMP_WARN:"Ambient"}-3`|WARNING|<p>**Depends on**:</p><p>- Ambient: Температура очень высокая: >{$TEMP_CRIT:"Ambient"}</p>|
|Ambient: Температура очень высокая: >{$TEMP_CRIT:"Ambient"}|<p>This trigger uses temperature sensor values as well as temperature sensor status if available</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}].avg(5m)}>{$TEMP_CRIT:"Ambient"}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}].max(5m)}<{$TEMP_CRIT:"Ambient"}-3`|HIGH||
|Ambient: Температура слишком низкая: <{$TEMP_CRIT_LOW:"Ambient"}|<p>-</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}].avg(5m)}<{$TEMP_CRIT_LOW:"Ambient"}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.Ambient.{#SNMPINDEX}].min(5m)}>{$TEMP_CRIT_LOW:"Ambient"}+3`|AVERAGE||
|CPU: Температура выше нормы: >{$TEMP_WARN:"CPU"}|<p>This trigger uses temperature sensor values as well as temperature sensor status if available</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.CPU.{#SNMPINDEX}].avg(5m)}>{$TEMP_WARN:"CPU"}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.CPU.{#SNMPINDEX}].max(5m)}<{$TEMP_WARN:"CPU"}-3`|WARNING|<p>**Depends on**:</p><p>- CPU: Температура очень высокая: >{$TEMP_CRIT:"CPU"}</p>|
|CPU: Температура очень высокая: >{$TEMP_CRIT:"CPU"}|<p>This trigger uses temperature sensor values as well as temperature sensor status if available</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.CPU.{#SNMPINDEX}].avg(5m)}>{$TEMP_CRIT:"CPU"}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.CPU.{#SNMPINDEX}].max(5m)}<{$TEMP_CRIT:"CPU"}-3`|HIGH||
|CPU: Температура слишком низкая: <{$TEMP_CRIT_LOW:"CPU"}|<p>-</p>|`{TEMPLATE_NAME:sensor.temp.value[tempReading.CPU.{#SNMPINDEX}].avg(5m)}<{$TEMP_CRIT_LOW:"CPU"}`<p>Recovery expression:</p>`{TEMPLATE_NAME:sensor.temp.value[tempReading.CPU.{#SNMPINDEX}].min(5m)}>{$TEMP_CRIT_LOW:"CPU"}+3`|AVERAGE||

## Feedback

Please report any issues with the template at https://support.zabbix.com

## Known Issues

- Description: Some IMMs (IMM1) do not return disks
  - Version: IMM1
  - Device: IBM x3250M3

- Description: Some IMMs (IMM1) do not return fan status: fanHealthStatus
  - Version: IMM1
  - Device: IBM x3250M3

- Description: IMM1 servers (M2, M3 generations) sysObjectID is NET-SNMP-MIB::netSnmpAgentOIDs.10
  - Version: IMM1
  - Device: IMM1 servers (M2,M3 generations)

- Description: IMM1 servers (M2, M3 generations) only Ambient temperature sensor available
  - Version: IMM1
  - Device: IMM1 servers (M2,M3 generations)
