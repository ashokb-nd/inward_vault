# Reprocessing Guide

## Default: Driver-i (bagheera2, US)

## Setup

```bash
# Clone repo
git clone https://github.com/netradyne/analytics.git
cd analytics
git checkout main
git submodule update --init --recursive

# Start docker (if not already running)
docker run --gpus all -it \
  -v /data4/ashok/REPROCESSING:/data4/ashok/REPROCESSING \
  -v /data4/ashok/REPROCESSING/autocam:/home/ubuntu/autocam \
  --name ashok_trt8 \
  --detach-keys="ctrl-p,ctrl-w" alert-kpi-python3-trt8:latest

# Or attach to existing container
docker attach ashok_trt8

# Inside docker
git config --global --add safe.directory '*'
umask 000
```

## Sync Alerts

```bash
cd /data4/ashok/REPROCESSING/analytics

mkdir -p ../alerts_dir/all

python dbutil/syncalert.py --avids <AVID> \
  --download-dir /data4/ashok/REPROCESSING/alerts_dir \
  --download-symlink-dir /data4/ashok/REPROCESSING/alerts_dir
```

## Reprocess

```bash
export PYTHONPATH=/data4/ashok/REPROCESSING/analytics:$PYTHONPATH

python src/reprocess.py --dnn-processing-mode ipe --dnn-runtime-library tensorrt \
  --outdir /data4/ashok/REPROCESSING/outdir/ \
  --basedir /data4/ashok/REPROCESSING/alerts_dir/all \
  --disable-inertial \
  --product-line bagheera2 --locale US \
  --alerts <AVID>
```

## DMS (use only if session is DMS)

```bash
# 1. Update config to enable DMS
# In nd_config.ini: [dms_drowsy] enabled = 1

# 2. Disable inward
# In nd_config.ini: [inward] enabled = 0

# 3. Fetch models for bagheera3
ak fetch_all_models

# 4. Run reprocess
export PYTHONPATH=/data4/ashok/REPROCESSING/analytics/src:$PYTHONPATH

python src/reprocess.py --dnn-processing-mode ipe --dnn-runtime-library tensorrt \
  --outdir /data4/ashok/REPROCESSING/DATA/outdirs_sp11/ \
  --basedir /data4/ashok/REPROCESSING/DATA/alerts_dir/all \
  --product-line bagheera3 \
  --disable-inward \
  --locale US \
  --alerts <AVID>
```

**Note:** If config changes don't take effect, run reprocessing twice instead of removing nd_config.ini.

## Verify Output

```bash
ls /data4/ashok/REPROCESSING/outdir/<AVID>/
# Expected files: summary.json, alerts.json, summary_ld.json, inward_cached.obsdata
```

## Common Errors

config.ini is created using the locale specific config and other overrides etc. 
and that is used further for reprocessing.
| Error | Fix |
|-------|-----|
| Config not updating | Run reprocessing twice |
| Stale nd_config.ini | Remove it and let it regenerate |
| Model not found | Run `ak fetch_all_models` |

