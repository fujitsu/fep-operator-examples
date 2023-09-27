# SampleCR for building within an intranet

## Assumed Scenario

This configuration is for users who want to use an environment in which a database is built within an intranet.  
Enable transparent data encryption to protect data from retrieval by disk references.  
MTLS provides secure data exchange by encrypting communication between containers.  

## Sample Custom Resource List

Lists the features that are enabled in the sample custom resource.

- FEPCluster Custom Resource: sample_fepcluster_cr.yaml
  - Consumed resources
    - core: 2 cores/1 instance
    - Memory: 8GB/1 instance
    - Instances: 3
  - Streaming Replication
  - Backup to a persistent volume
    - Every Sunday: Full backup
    - Daily: Incremental backup
  - Audit Log
  - MTLS
  - Transparent Data Encryption
  - Metrics Monitoring
  - Log forwarding (fluentbit)

- FEPLogging Custom Resource: sample_feplogging_cr.yaml
  - Log forwarding (fluentd)

- FEPPgpool2 Custom Resource: sample_feppgpool2_cr.yaml
  - connection pooling(pgpool2)

## How to Build a Sample Custom Resource

1. Sizing  
   This sample assumes a data storage capacity of 5GB, a daily operation time of 8 hours, and a daily update capacity of 1.5GB.  
   To change the amount of data or operation, edit the FEPCluster custom resource as described below.  

   - To increase the amount of data or the number of DB connections

      | path | description |
      | ---- | ----------- |
      | spec.fep.mcSpec                   | This defines the CPU and memory allocated to the container in which Postgres is running. |
      | spec.fepChildCrVal.customPgParams | The definition of postgres.conf. If you change the allocated memory, change the shared memory definition. Refer to "Fujitsu Enterprise Postgres Memory Requirements" in the [Fujitsu Enterprise Postgres Installation and Setup Guide for Server](https://www.postgresql.fastware.com/product-manuals)  for an estimate of shared memory, etc. |
      | spec.fepChildCrVal.storage | A definition of a persistent volume that stores data, logs, backups, etc. For disk size estimates, see "Estimating Database Disk Space Requirements" in the [Fujitsu Enterprise Postgres Installation and Setup Guide for Server](https://www.postgresql.fastware.com/product-manuals). |

1. Customizing the Sample Custom Resource  
  Here are some key edits to the sample custom resource.
  See the [Fujitsu Enterprise Postgres for Kubernetes Reference](https://www.postgresql.fastware.com/product-manuals) for a detailed description of the settings.

    - FEPClusterCR(sample_fepcluster_cr.yaml)

        | path                        | description |
        |------                       |------|
        | spec.fepChildCrVal.sysUsers | Change password of user to manage database |
        | spec.fepChildCrVal.storage  | Specify storageClass in the definition of each volume according to your environment. If storageClass is omitted, the default storage class is used.<br>backup and archivewalVol must be StorageClass with ReadWriteMany available in AccessModes |

1. Creating Certificates  
   Refer to "Manual Certificate Management" in the [Fujitsu Enterprise Postgres for Kubernetes User's Guide](https://www.postgresql.fastware.com/product-manuals) to create the certificate and private key for MTLS below.  
    
    | Custom Resource | parameters                         | description |
    | --------------- | ---------------------------------- | ----------- |
    | fepcluster      | spec.fepChildCrVal.sysUsers.xxxTls | Specify the Secret or ConfigMap containing the created certificate. |
    |                 | sepc.fep.patroni.tls               |  |
    |                 | spec.fep.postgres.tls              |  |
    |                 | spec.fep.monitoring.tls            |  |
    |                 | spec.fep.remoteLogging.tls         |  |
    | feplogging      | spec.fepLogging.tls                |  |
    | feppgpool2      | spec.customsslcert                 | For feppgpool2, put the certificate value in the CR. |
    |                 | spec.customsslkey                  |  |
    |                 | spec.customsslcacert               |  |

    In addition, create a client certificate and private key for MTLS communication between Pgpool2 and the client.

1. Apply Sample Custom Resource  
Once you've fixed the CustomResource and created the point of interest, apply the Sample custom resource to your Kubernetes environment.
Allow Pgpool2 connection in FEPClusterCR after FEPPgpool2CR application.
To allow TLS connections from the pgpool2 container, check the IP address of the pgpool2 container and modify FEPClusterCR as follows:
    ```
    $ kubectl get pod -o wide
    NAME                              READY   STATUS      RESTARTS        AGE    IP              
    new-fep-pgpool2-feppgpool2-0      1/1     Running            0        10m    10.0.0.xxx    
    new-fep-pgpool2-feppgpool2-1      1/1     Running            0        10m    10.0.0.yyy    

    $ kubectl edit fepcluster new-fep
        customPgHba: |
          # define pg_hba custom rules here to be merged with default rules.
          # If you use Pgpool2, set the METHOD to md5.
          # TYPE     DATABASE        USER        ADDRESS        METHOD
          hostssl    all             all         10.0.0.xxx     md5
          hostssl    all             all         10.0.0.yyy     md5
          hostssl    all             all         0.0.0.0/0      cert
          hostssl    replication     all         0.0.0.0/0      cert
    ```

1. How to connect from the Client  
   The client should be obtained from the [download site](https://www.postgresql.fastware.com/fujitsu-enterprise-postgres-client-download).
   To connect to PostgreSQL from a client, see "How to Connect to a FEP Cluster" in the [Fujitsu Enterprise Postgres for Kubernetes User's Guide](https://www.postgresql.fastware.com/product-manuals).
   The client certificate created by "Creating Certificates" can be specified in psql or as an environment variable, for example:
   See the PostgreSQL documentation for details.

    ```
    When specified with the psql command:
    $ psql "sslmode=require port=9995 host=new-pgpool2-feppgpool2-svc dbname=mydb user=postgres sslcert=~/client.crt sslkey=~/client.key sslrootcert=~/pgpool2server.crt" -c "SELECT 1"

    When specified in an environment variable
    $ export PGSSLMODE=require
    $ export PGSSLCERT=~/client.crt
    $ export PGSSLKEY=~/client.key
    $ export PGSSLROOTCERT=~/pgpool2server.crt
    ```