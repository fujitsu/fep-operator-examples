# Fujitsu Enterprise Postgres Operator Examples  

## Sample Custom Resources
This repository provides sample files that you can use to run Fujitsu Enterprise Postgres Operator.  
This repository contains various examples such as:.

- DefaultCR : sample yaml file for deploying a database cluster. These custom resources are minimum configuration.
- Monitoring : sample files for monitoring functions
- SampleCR_within_intranet : When building a database within an intranet
- SampleCR_using_external_cloud_services : When you want to achieve reliable security operations by linking with cloud services

In SampleCR_within_intranet and SampleCR_using_external_cloud_services folders, only TLS communication is allowed between FEPCluster and FEPPgpool2.  
Once you've fixed the CustomResource and created the point of interest, apply the Sample custom resource to your Kubernetes environment.  
Allow Pgpool2 connection in FEPClusterCR after FEPPgpool2CR application.  
For more information, please refer to "Apply Sample Custom Resource" of "How to Build a Sample Custom Resource" in README.md in each folder.

## Reference
Please refer to the following for the product website and manuals.  
Homepage https://www.postgresql.fastware.com/fujitsu-enterprise-postgres-for-kubernetes  
Document https://www.postgresql.fastware.com/product-manuals  
Product contact information pgtechenquiry@au.fujitsu.com