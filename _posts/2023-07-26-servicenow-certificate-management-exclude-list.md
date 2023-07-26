---
title: ServiceNow Certificate Management Exclude List
---

I recently had an IT team using Microsoft's PKI --> ServiceNow Certificate Management integration, ask if we could exclude a list of certificates that are frequently renewed, from tracking.

To accomplish this we can create a Discovery pre-post processor and system property.
# Steps

1.  Create new system property:  
    -   Name: "sn_disco_certmgmt.my.msca.ignorelist"
    -   type: string
    -   value: "tempcert1.domain.com,tempcert2.domain.com"
2.  Open Pattern Designer -> Pre Post Processing (sa_pattern_prepost_script)
3.  Create a new record  
    -   Name: MY- MSCA customizations
    -   Pattern/s: MicroSoft CA - Certificate Management
    -   When to execute: 1 - Pre sensor
    -   Script: (most of this was borrowed from OOB processor "Certificate Management - Adding SAN and Renewal Tracking")

<script src="https://gist.github.com/meatsac/abd617a3bdaa99232c04c8cf4f08424e.js"></script>
