# Apache Atlas Deployment with Ansible

This repository contains an Ansible playbook (`main.yml`) to deploy and configure Apache Atlas 2.4.0 with its dependencies (Kafka, Solr, HBase) on a multi-node cluster. The setup supports high availability (HA) for Atlas and integrates with pre-installed external services.

## Cluster Overview

- **Nodes**:
  - `master1` (192.168.22.10): Primary Atlas server, ZooKeeper quorum member, HBase master.
  - `master2` (192.168.22.11): Secondary Atlas server (HA), ZooKeeper quorum member.
  - `master3` (192.168.22.16): ZooKeeper quorum member.
  - `slave2` (192.168.22.14): Kafka broker, Solr instance.
  - `edge1` (192.168.22.17): Solr instance, potential edge node for clients/UI.

- **Assumptions**:
  - HBase, Kafka, and Solr are pre-installed at specified paths.
  - ZooKeeper quorum runs on `master1`, `master2`, and `master3` at port 2181.
  - SSH access is available as `admin` (adjustable in `inventory`).

## Components

- **Atlas**: Version 2.4.0, deployed on `master1` and `master2` with HA.
- **Kafka**: Version 2.11-2.2.0, on `slave2` (192.168.22.14:9092).
- **Solr**: Version 8.11.2, on `slave2` and `edge1` in cloud mode.
- **HBase**: Version 2.5.5-hadoop3, pre-installed, managed externally.
- **ZooKeeper**: Quorum at `192.168.22.10:2181,192.168.22.11:2181,192.168.22.16:2181`.

## Prerequisites

- Ansible 2.9+ installed on the control node.
- SSH access to all nodes with a common user (e.g., `admin`).
- Python 3 on all target nodes (`/usr/bin/python3`).
- Pre-installed dependencies (HBase, Kafka, Solr) at specified paths:
  - `/opt/hbase-2.5.5-hadoop3`
  - `/opt/kafka_2.11-2.2.0`
  - `/opt/solr-8.11.2`
