# chart-bulk-scan

Helm chart for Bulk Scan product

[![Build Status](https://dev.azure.com/hmcts/CNP/_apis/build/status/Helm%20Charts/chart-bulk-scan)](https://dev.azure.com/hmcts/CNP/_build/latest?definitionId=234)

This chart installs Bulk scan product ( Processor and Orchestractor ) with optional dependant services. 

## Usage FAQ

**Question:** How do I use bulk scan chart in my product chart/ application. 

**Answer:** Add bulk-scan chart to `requirements.yaml` and configure below global properties:
and you need to have "bsp-storage-account-creds" sealed secret under your namespace, ask BSP team to get support.
```yaml
global:
  idamApiUrl: "https://idam-api.aat.platform.hmcts.net"
  idamWebUrl: idam-web-public.aat.platform.hmcts.net
  ccdDataStoreUrl: "http://ccd-data-store"
````

##
**Q:** Can I enable and use CCD Chart that comes with Bulk scan chart

**A.** You can use it, but note that it's not fully configured and you need to override ccd config as per chart-ccd. 

##
**Q:** I have a postgresql dependency already , can i reuse it for bulkscan.
**A.** Yes, It is recommended to run with single postgresql installation for entire product with different databases. You need to add bulk_scan database.

```yaml
postgresql:
  initdbScripts:
    init.sql: |-
      CREATE DATABASE "bulk_scan" WITH OWNER = hmcts ENCODING = 'UTF-8' CONNECTION LIMIT = -1;
      CREATE DATABASE "draftstore" WITH OWNER = hmcts ENCODING = 'UTF-8' CONNECTION LIMIT = -1; #Needed if using draft store from bulkscan chart
      CREATE DATABASE "evidence" WITH OWNER = hmcts ENCODING = 'UTF-8' CONNECTION LIMIT = -1;  #Needed if using dm store from bulkscan chart
````
##
**Q:** I already have a dm-store installation in my product chart, how do i reuse it.
**A.** You need to override below environment variables with relevant dm-store url 

```yaml
bulk-scan-processor:
  java:
    environment:
      DOCUMENT_MANAGEMENT_URL: "{{ .Release.Name }}-dm-store"
bulk-scan-orchestrator:
  java:
    environment:
      DOCUMENT_MANAGEMENT_URL: "{{ .Release.Name }}-dm-store"
````
and disable the one shipped by dm-store setting `bulkscan.dmStore.enabled` to `false`

##
**Q:** I already have a s2s dependency already in my product chart, how do i reuse it.
**A.** You need to override below environment variables with relevant s2s url

```yaml
bulk-scan-processor:
  java:
    environment:
      S2S_URL: "{{ .Release.Name }}-s2s"
bulk-scan-orchestrator:
  java:
    environment:
       S2S_URL: "{{ .Release.Name }}-s2s"
dm-store:
  java:
    environment:
      IDAM_S2S_BASE_URI: '{{ .Release.Name }}-s2s'
````

Also, set below secrets to s2s installation:
```yaml
rpe-service-auth-provider:
  java:
    environment:
      MICROSERVICEKEYS_BULK_SCAN_PROCESSOR: AAAAAAAAAAAAAAAA
      MICROSERVICEKEYS_BULK_SCAN_ORCHESTRATOR: AAAAAAAAAAAAAAAA
````

and disable the one shipped by dm-store setting `bulkscan.s2s.enabled` to `false`


