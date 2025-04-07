Apache Atlas Deployment with Ansible
This repository contains an Ansible playbook (main.yml) to deploy and configure Apache Atlas 2.4.0 with its dependencies (Kafka, Solr, HBase) on a multi-node cluster. The setup supports high availability (HA) for Atlas and integrates with pre-installed external services.

Cluster Overview
Nodes:
master1 (192.168.22.10): Primary Atlas server, ZooKeeper quorum member, HBase master.
master2 (192.168.22.11): Secondary Atlas server (HA), ZooKeeper quorum member.
master3 (192.168.22.16): ZooKeeper quorum member.
slave2 (192.168.22.14): Kafka broker, Solr instance.
edge1 (192.168.22.17): Solr instance, potential edge node for clients/UI.
Assumptions:
HBase, Kafka, and Solr are pre-installed at specified paths.
ZooKeeper quorum runs on master1, master2, and master3 at port 2181.
SSH access is available as admin (adjustable in inventory).
Components
Atlas: Version 2.4.0, deployed on master1 and master2 with HA.
Kafka: Version 2.11-2.2.0, on slave2 (192.168.22.14:9092).
Solr: Version 8.11.2, on slave2 and edge1 in cloud mode.
HBase: Version 2.5.5-hadoop3, pre-installed, managed externally.
ZooKeeper: Quorum at 192.168.22.10:2181,192.168.22.11:2181,192.168.22.16:2181.
Prerequisites
Ansible 2.9+ installed on the control node.
SSH access to all nodes with a common user (e.g., admin).
Python 3 on all target nodes (/usr/bin/python3).
Pre-installed dependencies (HBase, Kafka, Solr) at specified paths:
/opt/hbase-2.5.5-hadoop3
/opt/kafka_2.11-2.2.0
/opt/solr-8.11.2


Playbook (main.yml)
The playbook automates:

Prerequisites: Installs required packages (maven, wget, etc.) and creates the hadoopZetta user.
Atlas: Builds and deploys Atlas on master1 and master2, configures HA, starts HBase on master1.
Solr: Configures and starts pre-installed Solr on slave2 and edge1, creates collections on slave2.
Kafka: Configures and starts pre-installed Kafka on slave2, creates Atlas topics (ATLAS_HOOK, ATLAS_ENTITIES).
Variables
atlas_version: "2.4.0"
atlas_install_dir: "/opt/apache-atlas-{{ atlas_version }}"
solr_version: "8.11.2"
solr_install_dir: "/opt/solr-{{ solr_version }}"
zookeeper_quorum: "192.168.22.10:2181,192.168.22.11:2181,192.168.22.16:2181"
kafka_broker: "192.168.22.14:9092"
atlas_user: "hadoopZetta"
hbase_dir: "/opt/hbase-2.5.5-hadoop3"
kafka_dir: "/opt/kafka_2.11-2.2.0"
Configuration Templates
atlas-application.properties.j2
Configures Atlas for HBase, Solr (cloud mode), Kafka, and HA.
Key settings:
atlas.kafka.bootstrap.servers=192.168.22.14:9092
atlas.graph.index.search.solr.zookeeper-url={{ zookeeper_quorum }}
atlas.server.ha.enabled=true
atlas-env.sh.j2
Sets environment variables for Atlas.
Key updates:
ATLAS_OPTS="-Djanusgraph.index.search.solr.http-urls=http://192.168.22.14:8983/solr,http://192.168.22.17:8983/solr"
MANAGE_LOCAL_HBASE=false, MANAGE_LOCAL_SOLR=false
server.properties.j2
Configures Kafka on slave2.
Key settings:
listeners=PLAINTEXT://192.168.22.14:9092
advertised.listeners=PLAINTEXT://192.168.22.14:9092
zookeeper.connect={{ zookeeper_quorum }}


Kafka Errors:
Verify topics: /opt/kafka_2.11-2.2.0/bin/kafka-topics.sh --describe --topic ATLAS_HOOK --bootstrap-server 192.168.22.14:9092.
Solr Issues:
Check Solr status: su - hadoopZetta -c "/opt/solr-8.11.2/bin/solr status".
Latest Updates
Kafka: Updated to 192.168.22.14:9092 (previously 192.168.22.13:9092) in server.properties.j2 and playbook vars.
Solr: Configured for slave2 (192.168.22.14) and edge1 (192.168.22.17), HTTP URLs in atlas-env.sh.j2.
Atlas HA: Enabled on master1 and master2 with consistent IDs and addresses.
Pre-installed Checks: Skips installation if HBase, Kafka, or Solr are present at expected paths.
Inventory: Added to define all nodes with ansible_user=admin.
Atlas Start Fix: Added DISPLAY="" in playbook to suppress X server errors.
Notes
Adjust memory settings (ATLAS_SERVER_HEAP) in atlas-env.sh.j2 if your nodes have limited RAM.
Ensure ZooKeeper is running before deployment.
For production, consider enabling SSL/TLS and Kerberos in atlas-application.properties.j2.
