# Apache Atlas Cluster Setup with Ansible

This repository contains an Ansible playbook (`playbook.yml`) and configuration templates to deploy Apache Atlas 2.4.0 with its dependencies (HBase, Solr, Kafka) across a cluster. The setup is tailored for a specific environment with `master1` (192.168.22.10), `master2` (192.168.22.11), and `slave2` (192.168.22.14), using `hadoopZetta` as the runtime user and `root` for installations.

## Cluster Overview
- **master1 (192.168.22.10)**: Runs Atlas (HA instance 1), HBase, Solr, and part of the ZooKeeper quorum.
- **master2 (192.168.22.11)**: Runs Atlas (HA instance 2) and part of the ZooKeeper quorum.
- **slave2 (192.168.22.14)**: Runs Kafka broker.
- **ZooKeeper Quorum**: Runs on `192.168.22.10:2181`, `192.168.22.11:2181`, `192.168.22.16:2181` (assumed pre-installed).

## Purpose
The goal is to automate the installation and configuration of Apache Atlas with high availability (HA), integrated with external HBase, Solr, and Kafka. The setup ensures:
- `root` performs all installations (since `hadoopZetta` lacks sudo privileges).
- `hadoopZetta` owns configuration files and runs services.

## What Was Done

### Playbook (`playbook.yml`)
- **Prerequisites**:
  - Creates the `hadoopZetta` user on all nodes.
  - Installs required packages (`maven`, `wget`, `tar`, `curl`, `openjdk-8-jdk`) as `root`.
- **Atlas on `master1` and `master2`**:
  - Downloads and builds Atlas 2.4.0 from source as `root`.
  - Installs binaries in `/opt/apache-atlas-2.4.0`.
  - Deploys templated config files (`atlas-application.properties`, `atlas-env.sh`) as `root`, then changes ownership to `hadoopZetta`.
  - Starts HBase on `master1` and Atlas on both nodes as `hadoopZetta`.
- **Solr on `master1`**:
  - Installs Solr 8.11.2 as `root` in `/opt/solr-8.11.2`.
  - Changes ownership to `hadoopZetta`.
  - Starts Solr in cloud mode and creates collections (`vertex_index`, `edge_index`, `fulltext_index`) as `hadoopZetta`.
- **Kafka on `slave2`**:
  - Installs Kafka 3.6.0 as `root` in `/opt/kafka`.
  - Deploys `server.properties` as `root`, then changes ownership to `hadoopZetta`.
  - Starts Kafka and creates topics (`ATLAS_HOOK`, `ATLAS_ENTITIES`) as `hadoopZetta`.

### Configuration Templates (in `templates/`)

#### `atlas-application.properties.j2`
- **Purpose**: Configures Atlas for HBase storage, Solr indexing, Kafka notifications, and HA.
- **Updates**:
  - Templated ZooKeeper quorum (`{{ zookeeper_quorum }}`) and Kafka broker (`{{ kafka_broker }}`).
  - Set `atlas.kafka.data` to `{{ atlas_install_dir }}/data/kafka`.
  - Set `atlas.authentication.method.file.filename` to `{{ atlas_install_dir }}/conf/users-credentials.properties`.
  - Kept HA addresses (`192.168.22.10:21000`, `192.168.22.11:21000`) hardcoded for `master1` and `master2`.
- **Deployed**: `/opt/apache-atlas-2.4.0/conf/atlas-application.properties`, owned by `hadoopZetta`.

#### `atlas-env.sh.j2`
- **Purpose**: Sets environment variables for Atlas runtime (e.g., Java options, paths).
- **Updates**:
  - Templated `ATLAS_HOME_DIR` as `{{ atlas_install_dir }}` (set to `/opt/apache-atlas-2.4.0`).
  - Kept `JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64` (adjust if needed).
  - Solr URL (`http://192.168.22.10:8983/solr`) remains hardcoded for `master1`.
- **Deployed**: `/opt/apache-atlas-2.4.0/conf/atlas-env.sh`, owned by `hadoopZetta`, executable.

#### `server.properties.j2`
- **Purpose**: Configures Kafka broker on `slave2`.
- **Updates**:
  - Templated `listeners` and `advertised.listeners` with `{{ kafka_broker }}` (set to `192.168.22.14:9092`).
  - Templated `log.dirs` with `{{ kafka_dir }}/logs` (set to `/opt/kafka/logs`).
  - Templated `zookeeper.connect` with `{{ zookeeper_quorum }}`.
- **Deployed**: `/opt/kafka/config/server.properties`, owned by `hadoopZetta`.

## Key Design Decisions
- **Root vs. `hadoopZetta`**:
  - `root` (via `become: yes`) installs software and deploys files since `hadoopZetta` isn’t a sudoer.
  - Ownership is changed to `hadoopZetta` post-installation, and services run as `hadoopZetta` (via `become_user`).
- **Installation Paths**: Everything in `/opt` (e.g., `/opt/apache-atlas-2.4.0`, `/opt/kafka`) for standard placement, manageable by `root`.
- **Dependencies**: Assumes ZooKeeper is pre-installed; HBase is pre-installed at `/opt/hbase-2.5.5-hadoop3` (add installation tasks if needed).

## Prerequisites
- **Ansible Inventory** (`ansible_hosts`):
  ```ini
  [master1]
  192.168.22.10 ansible_user=admin

  [master2]
  192.168.22.11 ansible_user=admin

  [slave2]
  192.168.22.14 ansible_user=admin


How to Run:
ansible-playbook -i ansible_hosts playbook.yml

---

### What This Covers
- **Overview**: Describes your cluster layout and goals.
- **Changes**: Details what I updated in the playbook and templates.
- **Instructions**: Provides steps to run and verify the setup.
- **Assumptions**: Notes dependencies and potential adjustments.

This `README.md` should give you a clear picture of the setup and how to use it. If you want to tweak it (e.g., add more details or sections), just say the word!
