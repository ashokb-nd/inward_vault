# Reprocessing guide

### 1. clone
- use your working branch in place of main.
```sh
git clone https://github.com/netradyne/analytics.git
cd analytics
git checkout main
git submodule update --init --recursive
```


### 2. fetch model weight
- fetch all models
    - change the config file, download paths, if necessary.
    - copy `fetch_all_models.py` file to basedir
    - ensure that the models are downloaded to the location that's mapped to "/home/ubuntu/autocam" of docker.

```sh
#ak is my custom python module. it can be used as command
# it's installed in conda env named 'ashok'. (it's outside the docker.)
# this could be done before attaching the docker.
ak fetch_all_models
# there are arguments to choose to download models of a particular config based on the choise of locale, product-line
```

### 3. reprocess

get into docker
- create a docker container

if it's already there. attach to it. I use the name `ashok_trt8

```sh
docker attach ashok_trt8
```

else start a new one
my code base is at 'REPROCESSING' folder.
```bash
docker run --gpus all -it \
-v /data4/ashok/REPROCESSING:/data4/ashok/REPROCESSING \
-v /data4/ashok/REPROCESSING/autocam:/home/ubuntu/autocam \
--name ashok_trt8 \
--detach-keys="ctrl-p,ctrl-w" alert-kpi-python3-trt8:latest
```

```bash
#inside docker 
git config --global --add safe.directory '*'
umask 000 

```

#### 3a. build

```bash
make clean
make -j48  all PYTHON3=1 PROC_ENV_KPI=1 REPROCESS=1
```
#### 3b. sync alerts

run these from "./analytics/" dir
- `88a6aae1-1f9c-43bc-90be-a2ef32c1cdec` is an example avid.
```bash
cd /data4/ashok/REPROCESSING/analytics

mkdir -p ../alerts_dir/all

python dbutil/syncalert.py --avids <AVID> \  # or --alerts <ALERT_ID>
# Note: folder name in basedir will be aavid (if using --avids) or aaid (if using --alerts)
#       Check syncalert output to see the folder name created
  --download-dir /data4/ashok/REPROCESSING/alerts_dir \
  --download-symlink-dir /data4/ashok/REPROCESSING/alerts_dir
```

---

### General instructions

- Driver-i, DMS are two different lines of development. our task involves working either one of these.
- explore the "help" of the reprocess.py for the arguments usage.
- when we say 'metadata reprocessing', it means to use "--dnn-processing-mode metadata"
- alerts argument is the name of the folder of the session data downloaded using dbutil/syncalert.py. for avid, it's avid only. but if an alert_id is given to syncalert it's folder name would be an 'aaid'. which is logged to terminal. 

- Driver-i, Bagheera2, US are defaults
- for dnn-processing-mode default is 'ipe'
### Driver-i

```bash
export PYTHONPATH=/data4/ashok/REPROCESSING/analytics:$PYTHONPATH

python src/reprocess.py --dnn-processing-mode ipe --dnn-runtime-library tensorrt \
  --outdir /data4/ashok/REPROCESSING/outdir/ \
  --basedir /data4/ashok/REPROCESSING/alerts_dir/all \
  --disable-inertial \
  --product-line bagheera2 --locale US \
  --alerts <FOLDER_NAME>  # folder name from syncalert (aavid if --avids, aaid if --alerts)
```

# DMS

For DMS, The following steps are necessary

1. update the config to enable DMS :

```c
[dms_drowsy]
enabled = 1
```

2. disable inward. ( we are not using it).
3. --dnn-processing-mode = ipe . (doesn't work if metadata)
4. use bagheera3 
5. (optional, only if faces some issue with it) remove nd_config.ini file (sometimes it won't update. stale values are used.) -> don't remove. just run reprocessing twice. 
6. fetch-all models from that config

```sh
ak fetch_all_models
```

```sh
export PYTHONPATH=/data4/ashok/REPROCESSING/analytics/src:$PYTHONPATH
python  src/reprocess.py --dnn-processing-mode ipe --dnn-runtime-library tensorrt \
                        --outdir /data4/ashok/REPROCESSING/DATA/outdirs/ \
                        --basedir /data4/ashok/REPROCESSING/DATA/alerts_dir/all \
                        --product-line bagheera3 \
                        --disable-inward \
                        --locale US \
                        --alerts 8a122ec9-0949-40ac-a9e2-3a0923151403
```


## Verify Output

```shell
ls /data4/ashok/REPROCESSING/DATA/outdir/<FOLDER_NAME>/
# Expected files: summary.json, alerts.json, summary_ld.json, inward_cached.obsdata
```

