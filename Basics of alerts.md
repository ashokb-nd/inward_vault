# Basic terminology used.

The company is "Netradyne".

a session is one minute of the driving. we use alert ids, avids for reprocessing. 
- alert_id is an integer : ex: 5509310223
- avid is alpha numeric string like this : 8f411098-5459-4510-8d83-74a4e78ae8c0

reprocessing a way to emulate the driver monitoring system happening on the edge device, in EC2. the process in high level is this. 
1. download the videos, metadata (the file containing all the relevent detections info, results etc.)
2. reprocess that alerts/ avids.
3. check the output dir for results. often the "summary.json" file there is of main interest. to check if certain alerts are raised/suppressed etc. 

Multiple configs
- there are multiple config files depending on the region and versions of models offering by the company.
- each config corresponds to a combination of product and locale.

# EC2 related
we use an EC2 for our work. it is shared by our team.
I use the following
- conda env : 'ashok'

### Usage of screen 
- I use screen to run detachable scripts. my reprocessing often goes like this
	- make a screen with name 'ashok_trt' 
	- attach the docker inside the screen 
	- run the reprocessing. it often takes time so detach if needed. 
	- and keep the screen. use it again if i need to reprocess in future.
