## Problem
Root filesystem usage crossed 70% on Kubernetes worker nodes.

## Detection
Checked disk usage using Ansible.

## Analysis
Disk usage was normal OS growth (swap, snap, /usr).

## Decision
Chose disk expansion instead of risky cleanup.

## Fix
Resized VM disk and expanded LVM volumes.

## Result
Disk usage reduced with no downtime.

## Learning
Always analyze before cleaning disk.
