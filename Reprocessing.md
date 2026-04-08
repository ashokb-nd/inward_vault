# Simple Reprocess

clone
- use your working branch in place of main.
```sh
git clone https://github.com/netradyne/analytics.git
cd analytics
git checkout main
git submodule update --init --recursive
```

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

Run it second time if it doesn't work for the first time.

```bash
make clean
make -j48  all PYTHON3=1 PROC_ENV_KPI=1 REPROCESS=1
```

- fetch all models
    - change the config file, download paths, if necessary.
    - copy `fetch_all_models.py` file to basedir
    - ensure that the models are downloaded to the location that's mapped to "/home/ubuntu/autocam" of docker.

```sh
#ak is my custom python module. it can be used as command
# it's installed in conda env named 'ashok'. (it's outside the docker.)
# this could be done before attaching the docker.
ak fetch_all_models
```

sync alerts  
run these from "./analytics/" dir
- `88a6aae1-1f9c-43bc-90be-a2ef32c1cdec` is an example avid.
```bash
mkdir -p ../alerts_dir/all

python dbutil/syncalert.py --avids 88a6aae1-1f9c-43bc-90be-a2ef32c1cdec  \
--download-dir /data4/ashok/REPROCESSING/alerts_dir \
--download-symlink-dir /data4/ashok/REPROCESSING/alerts_dir
```

```bash
export PYTHONPATH=/data4/ashok/REPROCESSING/analytics:$PYTHONPATH
cd /data4/ashok/REPROCESSING/analytics

```


- Driver-i, DMS are two different lines of development. our task involves working either one of these.
- explore the "help" of the reprocess.py for the arguments usage.
- when we say 'metadata reprocessing', it means to use "--dnn-processing-mode metadata"
- alerts argument is the name of the folder of the session data downloaded using dbutil/syncalert.py. for avid, it's avid only. but if an alert_id is given to syncalert it's folder name would be an 'aaid'. which is logged to terminal. 
### Driver-i

```bash
python  src/reprocess.py --dnn-processing-mode ipe --dnn-runtime-library tensorrt \
                        --outdir /data4/ashok/REPROCESSING/outdir/ \
                        --basedir /data4/ashok/REPROCESSING/alerts_dir/all \
                        --disable-inertial \
                        --product-line bagheera2 --locale US \
                        --alerts 88a6aae1-1f9c-43bc-90be-a2ef32c1cdec \
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
                        --outdir /data4/ashok/REPROCESSING/DATA/outdirs_sp11/ \
                        --basedir /data4/ashok/REPROCESSING/DATA/alerts_dir/all \
                        --product-line bagheera3 \
                        --disable-inward \
                        --locale US \
                        --alerts 8a122ec9-0949-40ac-a9e2-3a0923151403
```

