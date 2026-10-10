# OCI Roadmap 문서 연결 목록

현재 93개 학습항목에 대응하는 **작성예정 초안**입니다. 실제 실습 본문과 OCI 실행 검증은 포함하지 않습니다. 사용자 검토와 수정 요청을 거쳐 후속 작성합니다.

## 번호 규칙

상위 디렉터리는 `00_`부터 연속 번호를 사용합니다. 각 디렉터리의 상위 항목 문서는 `00_`이며 하위 항목은 `01_`부터 연속 번호입니다. Roadmap의 세로 위치와 같은 행의 좌→우 배치를 기준으로 그룹을 정렬했습니다. 병렬 그룹을 직렬 필수 학습 단계로 해석하지 마세요. 기존 Node ID와 영문 항목명은 변경하지 않았습니다.

## 공통 작성 양식

[공통 템플릿](TEMPLATE.md) · [작성 및 검토 규칙](CONTRIBUTING.md) · [JSON 연결 매핑](roadmap-mapping.json) · [CSV 연결 매핑](roadmap-mapping.csv)

## 전체 항목

### 00 Introduction

- [00 Introduction](00_Introduction/00_Introduction.md) — `sequence-0-0`
- [01 Cloud Computing](00_Introduction/01_Cloud_Computing.md) — `sequence-0-1`
- [02 IaaS / PaaS / SaaS](00_Introduction/02_IaaS_PaaS_SaaS.md) — `sequence-0-2`
- [03 Cloud Deployment Models](00_Introduction/03_Cloud_Deployment_Models.md) — `sequence-0-3`

### 01 Introduction to OCI

- [00 Introduction to OCI](01_Introduction_to_OCI/00_Introduction_to_OCI.md) — `sequence-1-0`
- [01 Regions](01_Introduction_to_OCI/01_Regions.md) — `sequence-1-1`
- [02 Availability Domains](01_Introduction_to_OCI/02_Availability_Domains.md) — `sequence-1-2`
- [03 Fault Domains](01_Introduction_to_OCI/03_Fault_Domains.md) — `sequence-1-3`
- [04 Shared Security Model](01_Introduction_to_OCI/04_Shared_Security_Model.md) — `sequence-1-4`
- [05 Well-Architected Framework](01_Introduction_to_OCI/05_Well_Architected_Framework.md) — `sequence-1-5`

### 02 Compute

- [00 Compute](02_Compute/00_Compute.md) — `sequence-2-0`
- [01 Shapes](02_Compute/01_Shapes.md) — `sequence-2-1`
- [02 Burstable Instances](02_Compute/02_Burstable_Instances.md) — `sequence-2-2`
- [03 Boot Volume](02_Compute/03_Boot_Volume.md) — `sequence-2-3`
- [04 Block Volume](02_Compute/04_Block_Volume.md) — `sequence-2-4`
- [05 SSH Keys](02_Compute/05_SSH_Keys.md) — `sequence-2-5`
- [06 Public IP Addresses](02_Compute/06_Public_IP_Addresses.md) — `sequence-2-6`
- [07 Cloud-init / User Data](02_Compute/07_Cloud_init_User_Data.md) — `sequence-2-7`
- [08 Capacity Types](02_Compute/08_Capacity_Types.md) — `sequence-2-8`

### 03 Virtual Cloud Network (VCN)

- [00 Virtual Cloud Network (VCN)](03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md) — `sequence-3-0`
- [01 CIDR Blocks](03_Virtual_Cloud_Network_VCN/01_CIDR_Blocks.md) — `sequence-3-1`
- [02 Subnets](03_Virtual_Cloud_Network_VCN/02_Subnets.md) — `sequence-3-2`
- [03 Private Subnets](03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md) — `sequence-3-3`
- [04 Public Subnets](03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md) — `sequence-3-4`
- [05 Route Tables](03_Virtual_Cloud_Network_VCN/05_Route_Tables.md) — `sequence-3-5`
- [06 Network Security Groups](03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md) — `sequence-3-6`
- [07 Security Lists](03_Virtual_Cloud_Network_VCN/07_Security_Lists.md) — `sequence-3-7`
- [08 Ingress / Egress](03_Virtual_Cloud_Network_VCN/08_Ingress_Egress.md) — `sequence-3-8`
- [09 Stateful / Stateless Rules](03_Virtual_Cloud_Network_VCN/09_Stateful_Stateless_Rules.md) — `sequence-3-9`
- [10 Internet Gateway](03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md) — `sequence-3-10`
- [11 NAT Gateway](03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md) — `sequence-3-11`
- [12 Service Gateway](03_Virtual_Cloud_Network_VCN/12_Service_Gateway.md) — `sequence-3-12`
- [13 VNICs](03_Virtual_Cloud_Network_VCN/13_VNICs.md) — `sequence-3-13`

### 04 Identity and Access Management (IAM)

- [00 Identity and Access Management (IAM)](04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) — `sequence-4-0`
- [01 Users](04_Identity_and_Access_Management_IAM/01_Users.md) — `sequence-4-1`
- [02 Groups](04_Identity_and_Access_Management_IAM/02_Groups.md) — `sequence-4-2`
- [03 Dynamic Groups](04_Identity_and_Access_Management_IAM/03_Dynamic_Groups.md) — `sequence-4-3`
- [04 Policies](04_Identity_and_Access_Management_IAM/04_Policies.md) — `sequence-4-4`
- [05 Compartments](04_Identity_and_Access_Management_IAM/05_Compartments.md) — `sequence-4-5`
- [06 OCID](04_Identity_and_Access_Management_IAM/06_OCID.md) — `sequence-4-6`
- [07 Policy Conditions](04_Identity_and_Access_Management_IAM/07_Policy_Conditions.md) — `sequence-4-7`
- [08 Policy Scope / Inheritance](04_Identity_and_Access_Management_IAM/08_Policy_Scope_Inheritance.md) — `sequence-4-8`
- [09 Resource Principals](04_Identity_and_Access_Management_IAM/09_Resource_Principals.md) — `sequence-4-9`

### 05 File Storage

- [00 File Storage](05_File_Storage/00_File_Storage.md) — `service-file-0`
- [01 Mount Targets](05_File_Storage/01_Mount_Targets.md) — `service-file-1`
- [02 Exports](05_File_Storage/02_Exports.md) — `service-file-2`
- [03 Export Options](05_File_Storage/03_Export_Options.md) — `service-file-3`
- [04 Snapshots](05_File_Storage/04_Snapshots.md) — `service-file-4`

### 06 Object Storage

- [00 Object Storage](06_Object_Storage/00_Object_Storage.md) — `sequence-6-0`
- [01 Buckets / Objects](06_Object_Storage/01_Buckets_Objects.md) — `sequence-6-1`
- [02 Object Lifecycle Management](06_Object_Storage/02_Object_Lifecycle_Management.md) — `sequence-6-2`
- [03 Storage Tiers](06_Object_Storage/03_Storage_Tiers.md) — `sequence-6-3`
- [04 Archive](06_Object_Storage/04_Archive.md) — `sequence-6-4`
- [05 Standard Storage](06_Object_Storage/05_Standard_Storage.md) — `sequence-6-5`
- [06 Infrequent Access](06_Object_Storage/06_Infrequent_Access.md) — `sequence-6-6`

### 07 Autoscaling

- [00 Autoscaling](07_Autoscaling/00_Autoscaling.md) — `sequence-7-0`
- [01 Images](07_Autoscaling/01_Images.md) — `sequence-7-1`
- [02 Instance Configurations](07_Autoscaling/02_Instance_Configurations.md) — `sequence-7-2`
- [03 Instance Pools](07_Autoscaling/03_Instance_Pools.md) — `sequence-7-3`
- [04 Metric-based Autoscaling](07_Autoscaling/04_Metric_based_Autoscaling.md) — `sequence-7-4`
- [05 Schedule-based Autoscaling](07_Autoscaling/05_Schedule_based_Autoscaling.md) — `sequence-7-5`

### 08 Load Balancer

- [00 Load Balancer](08_Load_Balancer/00_Load_Balancer.md) — `service-lb-0`
- [01 Listeners](08_Load_Balancer/01_Listeners.md) — `service-lb-1`
- [02 Backend Sets](08_Load_Balancer/02_Backend_Sets.md) — `service-lb-2`
- [03 Health Checks](08_Load_Balancer/03_Health_Checks.md) — `service-lb-3`
- [04 SSL Certificates](08_Load_Balancer/04_SSL_Certificates.md) — `service-lb-4`

### 09 Network Load Balancer

- [00 Network Load Balancer](09_Network_Load_Balancer/00_Network_Load_Balancer.md) — `service-nlb-0`
- [01 Layer 3 / Layer 4](09_Network_Load_Balancer/01_Layer_3_Layer_4.md) — `service-nlb-1`
- [02 IP Address Preservation](09_Network_Load_Balancer/02_IP_Address_Preservation.md) — `service-nlb-2`
- [03 Health Check Policies](09_Network_Load_Balancer/03_Health_Check_Policies.md) — `service-nlb-3`
- [04 Long-running Connections](09_Network_Load_Balancer/04_Long_running_Connections.md) — `service-nlb-4`

### 10 Web Application Firewall (WAF)

- [00 Web Application Firewall (WAF)](10_Web_Application_Firewall_WAF/00_Web_Application_Firewall_WAF.md) — `service-waf-0`
- [01 WAF Policies](10_Web_Application_Firewall_WAF/01_WAF_Policies.md) — `service-waf-1`
- [02 Request Protection Rules](10_Web_Application_Firewall_WAF/02_Request_Protection_Rules.md) — `service-waf-2`
- [03 Access Control](10_Web_Application_Firewall_WAF/03_Access_Control.md) — `service-waf-3`
- [04 Rate Limiting](10_Web_Application_Firewall_WAF/04_Rate_Limiting.md) — `service-waf-4`

### 11 DNS

- [00 DNS](11_DNS/00_DNS.md) — `sequence-8-0`
- [01 DNS Zones / Records](11_DNS/01_DNS_Zones_Records.md) — `sequence-8-1`
- [02 Traffic Management](11_DNS/02_Traffic_Management.md) — `sequence-8-2`
- [03 DNS Health Checks](11_DNS/03_DNS_Health_Checks.md) — `sequence-8-3`

### 12 Monitoring

- [00 Monitoring](12_Monitoring/00_Monitoring.md) — `sequence-9-0`
- [01 Metrics](12_Monitoring/01_Metrics.md) — `sequence-9-1`
- [02 Alarms](12_Monitoring/02_Alarms.md) — `sequence-9-2`
- [03 Events](12_Monitoring/03_Events.md) — `sequence-9-3`
- [04 Logging](12_Monitoring/04_Logging.md) — `sequence-9-4`
- [05 Audit](12_Monitoring/05_Audit.md) — `sequence-9-5`
- [06 Notifications](12_Monitoring/06_Notifications.md) — `sequence-9-6`

### 13 Base Database Service

- [00 Base Database Service](13_Base_Database_Service/00_Base_Database_Service.md) — `sequence-10-0`
- [01 DB Systems](13_Base_Database_Service/01_DB_Systems.md) — `sequence-10-1`
- [02 Database Storage](13_Base_Database_Service/02_Database_Storage.md) — `sequence-10-2`
- [03 Backup / Restore](13_Base_Database_Service/03_Backup_Restore.md) — `sequence-10-3`
- [04 ASM](13_Base_Database_Service/04_ASM.md) — `sequence-10-4`
- [05 LVM](13_Base_Database_Service/05_LVM.md) — `sequence-10-5`
