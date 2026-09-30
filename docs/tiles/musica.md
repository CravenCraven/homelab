# Música (Navidrome)

Self-hosted music library for Laboratório Brasil. Runs Navidrome on k3s, managed by Flux.

- **Manifest:** `clusters/staging/brasil/navidrome.yaml`
- **Namespace:** `brasil`
- **Node:** `dell-node` (pinned by its `local-path` volumes)
- **Status:** running, tested, reachable on the LAN only

## How it's built

| Piece | Value |
|---|---|
| Container port | 4533 |
| Service | `navidrome`, ClusterIP, port 80, targetPort 4533 |
| Ingress | `musica.brasil.local` (Traefik, LAN only) |
| Music volume | PVC `navidrome-music`, 100Gi, mounted at `/music` |
| Data volume | PVC `navidrome-data`, 5Gi, mounted at `/data` (database, cache) |
| Storage class | `local-path` (node-pinned, can't be resized) |

### Environment variables

| Variable | Value | Why |
|---|---|---|
| `ND_MUSICFOLDER` | `/music` | Where the library lives |
| `ND_DATAFOLDER` | `/data` | Database and cache |
| `ND_SCANNER_SCHEDULE` | `0 3 * * *` | Daily scan. The container clock is UTC, so this is 3am UTC |
| `ND_SCANNER_SCANONSTARTUP` | `true` | Finds existing files on a fresh deploy |
| `ND_LOGLEVEL` | `info` | |

> Older Navidrome versions used `ND_SCANSCHEDULE` and `ND_ENABLESTARTUPSCAN`. The current version silently ignores those names. If the logs say `Periodic scan is DISABLED`, check the variable names first.

## Prerequisites

- `kubectl` access to the cluster from the machine running the browser
- SSH access to `dell-node` with sudo
- Music files you own (MP3, FLAC, M4A). Apple Music subscription downloads are DRM-locked and won't play

## 1. Verify it's running

```
kubectl -n brasil get pods,svc,ingress | grep -i navidrome
kubectl -n brasil logs deploy/navidrome | grep -i scan
```

Expect:

- Pod `1/1 Running`
- Service on `80/TCP`
- A log line: `Scheduling periodic scan schedule="0 3 * * *"`

## 2. Change configuration

All changes go through Git. Flux owns the manifest, so `kubectl edit` changes get reverted.

```
# edit clusters/staging/brasil/navidrome.yaml
git diff                      # confirm only the intended lines changed
git add clusters/staging/brasil/navidrome.yaml
git commit -m "fix(navidrome): <what changed>"
git push origin main
flux reconcile kustomization flux-system --with-source
```

Changing an env var changes the pod template, so a new pod replaces the old one.

## 3. Load music

The source of truth is one folder on the Mac: `~/Music/brasil`, organized as `Artist/Album/tracks`. Add music there only. Never edit the copy on the node by hand.

### Find the volume on disk

```
kubectl -n brasil get pvc navidrome-music -o jsonpath='{.spec.volumeName}{"\n"}'
kubectl get pv <pv-name> -o jsonpath='{.spec.hostPath.path}{.spec.local.path}{"\n"}'
kubectl get pv <pv-name> -o jsonpath='{.spec.nodeAffinity.required.nodeSelectorTerms[0].matchExpressions[0].values[0]}{"\n"}'
```

The first gives the PV name, the second the path, the third the node.

### Copy in two hops

The volume directory is owned by root. `rsync --rsync-path="sudo rsync"` fails because sudo can't prompt for a password over a non-interactive SSH session. Copy through your home directory instead:

```
# From the Mac: into your home folder on the node (no sudo needed)
rsync -avh ~/Music/brasil/ dell:~/navidrome-upload/

# On the node: into the volume
ssh dell
sudo rsync -avh ~/navidrome-upload/ <volume-path>/
sudo ls <volume-path>/
rm -rf ~/navidrome-upload
exit
```

Keep the trailing `/` on the source path. Without it, rsync copies the folder itself and the library ends up nested at `/music/brasil/...`.

Add `-n` to either rsync for a dry run first.

### Scan

The startup scan and the nightly scan pick up new files. To scan right away, use the activity icon (top right) in the web UI. The magnifier icon runs a full scan.

## 4. Test

Run the port-forward on the machine with the browser:

```
kubectl -n brasil port-forward svc/navidrome 4533:80
```

Open `http://localhost:4533`, then check:

- [ ] Log in. On a fresh install, the first login creates the admin account, so do this before anyone else on the LAN can
- [ ] Albums show up
- [ ] A song plays all the way through
- [ ] The library survives a restart:
  ```
  kubectl -n brasil rollout restart deploy/navidrome
  kubectl -n brasil get pods | grep navidrome     # new pod name, low AGE
  ```
  Restart the port-forward (it dies with the old pod), refresh, and confirm the albums and your login are still there

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Periodic scan is DISABLED` in logs | Old env var names | Use `ND_SCANNER_*` names |
| Scan finishes in a few ms | `/music` is empty | Check the rsync landed in the right PVC |
| `ERR_CONNECTION_REFUSED` | Port-forward died (usually after a restart) | `Ctrl+C` and rerun it |
| `ERR_CONNECTION_RESET` | Port-forward running on a different machine than the browser | Run it where the browser is |
| `broken pipe` in port-forward output | Browser closed a connection early | Harmless if the app works |
| `sudo: a terminal is required` during rsync | sudo can't prompt over non-interactive SSH | Use the two-hop copy |

## Next

- Expose publicly through Cloudflare Tunnel, behind Cloudflare Access
- Optional: `TZ=America/New_York` so the scan schedule runs on local time
