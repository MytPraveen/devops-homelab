Perfect 👍
Below is ONLY the content for disk-pressure-k3s-workers.md.
You can copy–paste directly into GitHub or Obsidian.

⸻


# Disk Pressure Handling on k3s Worker Nodes

## Context
- Environment: Personal DevOps homelab (24/7)
- Hypervisor: Proxmox
- OS: Ubuntu
- Cluster: Kubernetes (k3s)
- Storage: LVM
- Nodes affected: k3s worker nodes

> Note: This activity was performed in a personal homelab for learning purposes.

---

## Problem
Root filesystem usage on Kubernetes worker nodes crossed ~70%, which could lead to:
- Pod scheduling issues
- Node instability
- Disk pressure alerts

---

## Detection
Disk usage was checked centrally using Ansible:

```bash
ansible k3s_workers -m shell -a "df -h /"

Sample output:

/dev/mapper/ubuntu--vg-ubuntu--lv   19G   13G   5G   72% /


⸻

Analysis

To identify the cause of disk usage, directory-level analysis was performed:

ansible k3s_workers -m shell -a "du -sh /* 2>/dev/null | sort -h"

Findings:
	•	Disk usage was due to normal OS components such as:
	•	/usr
	•	/var
	•	snap
	•	No abnormal log growth or container data issues were observed

Conclusion:
	•	Cleanup would be temporary and risk recurring disk pressure

⸻

Decision

Chose disk expansion instead of deleting files:
	•	Safer long-term solution
	•	Avoids repeated alerts
	•	Aligns with production best practices

⸻

Fix

1. Resize VM disk (Proxmox)
	•	Increased VM disk size from 20G to 40G

2. Verify disk inside the VM

lsblk

3. Resize LVM physical volume

sudo pvresize /dev/sda3

4. Extend LVM logical volume

sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv

5. Resize filesystem (online)

sudo resize2fs /dev/ubuntu-vg/ubuntu-lv

6. Verify final disk usage

df -h /

Result:

/dev/mapper/ubuntu--vg-ubuntu--lv   37G   11G   24G   32% /


⸻

Result
	•	Root filesystem expanded successfully
	•	Disk usage reduced from ~70% to ~30–35%
	•	No downtime
	•	Kubernetes workloads remained unaffected

⸻

Learnings
	•	Disk expansion involves both hypervisor and OS-level changes
	•	LVM provides flexibility but requires additional resize steps
	•	Always analyze disk usage before choosing cleanup or expansion
	•	Disk expansion is often safer than deletion in production-like environments

---