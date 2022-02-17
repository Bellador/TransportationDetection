# Project to detect transportation modes in video footage
also copied to README.md

## My resources

- written project proposal: `"C:\Users\mhartman\Documents\PhD\Doktor Arbeit\Notes\20210415_indicator_design_proposal.md"`
- python project: `"C:\Users\mhartman\PycharmProjects\transportation_mode_detection_Yolov5"`
- ANACONDA ENV: `conda activate transportation_Yolov5_env` - contains everything for object detection, tracking and OCR


From project folder `Yolov5_DeepSort_Pytorch` all files were copied to `transportation_mode_detection_Yolov5`; file `track.py`was changed for our needs, especially to work without command line. Also class_names tracking was added to link tracks to class_ids. (COPYIED MANY DEEP SORT CORE ELEMENTS THAT NEEDED TO BE CHANGED FROM THE transportation_mode_detection_YoloV4 VERSION - SEEMED LIKE IT WAS DELIBERATLY LEFT OUT OR SOMETHING SMH...) 


## TODO
OCR was implemented on every video frame and logs for object tracking and OCR (e.g. detected street names) were implemented. - DONE!

- Analysing and matching OCR output to place names in research area with [Levenshtein Distance (word similarity)](https://towardsdatascience.com/calculating-string-similarity-in-python-276e18a7d33a) on OSM street name set as gazzetteer. - DONE!

Method 1 - Tried but issues to easily get all streets in the layers... (not used - see Method 2)

Presumably downloading a country [OSM dump e.g. Switzerland](https://download.geofabrik.de/europe/switzerland.html) which can then be imported into a local PostGIS enabled PostgresDB with the help of [osm2pgsql](https://osm2pgsql.org/) which was installed under the following path `"C:\Program Files\osm2pgsql-latest-x64"` also added to PATH as `osm2pgsql`

- OSM database with imported osm.pbf file for Switzerland was done with the following [manual](https://learnosm.org/en/osm-data/osm2pgsql/) and the custom cmd command `osm2pgsql -c -d osm -U postgres -W -H 127.0.0.1 -P 5433 -E 4326 -S default.style switzerland-latest.osm.pbf`. Flag `-E` specifies to which ESPG (SRID) the data is projected to!

- Import Geneva SHP file into db using shp2pgsql-gui.exe `C:\Program Files\PostgreSQL\13\bin\postgisgui\shp2pgsql-gui.exe`.


Method 2 - Functioned easily

- download OSM data for Geneva streets with the following Overpass API script:
```
[out:json][timeout:25];
// gather results
area["name"="Genève"];
// query part for: “highway=* and name=*”
way["highway"]["name"](area);

// print results
out body;
>;
out skel qt;
```
- export as geojson
- import geojson into Postgres with ogr2ogr:
`ogr2ogr -f "PostgreSQL" PG:"dbname=osm user=postgres password=XXXX port=5433" "C:\Users\mhartman\PycharmProjects\transportation_mode_detection_Yolov5\geoparsing\geneva_streets.geojson"` - easy! `-f` format


## Run code

`conda activate transportation_Yolov5_env`

`cd "C:\Users\mhartman\PycharmProjects\transportation_mode_detection_Yolov5"`

`python object_tracker.py --source 0 --yolo_weights yolov5s.pt --img 640 --show-vid --save-txt`

## External resources

The workflow is based on object (1) detection and (2) tracking as well as (3) Optical Character Recognition (OCR). These resources were taken from:

- (1) YOLOv5 [Github](https://github.com/mikel-brostrom/Yolov5_DeepSort_Pytorch/)
- (2) DeepSort [DeepSort](https://github.com/nwojke/deep_sort)
- (3) [EasyOCR](https://pypi.org/project/easyocr/)



# Serve Tensorflow street sign detection (FSNS) .pb model with Docker (DEAD END!)

resources:
- [FSNS Github](https://github.com/tensorflow/models/blob/master/research/attention_ocr/README.md#dataset)
- [Tensorflow Serving](https://www.tensorflow.org/tfx/serving/serving_basic)
- [Tensorflow Docker](https://www.tensorflow.org/tfx/serving/docker)

1. export model

`python model_export.py \
  --checkpoint=model.ckpt-399731 \
  --export_dir=./transportation_mode_detection_Yolov5\attention_ocr_fsns\fsns_attention_model`

2. get tensorflow serving docker image

`docker pull tensorflow/serving` (CPU)
`docker pull tensorflow/serving:latest-gpu` (GPU)

3. spin up tensorflow serving container
`docker run -p 8501:8501 --mount type=bind,source="C:\Users\mhartman\PycharmProjects\transportation_mode_detection_Yolov5\attention_ocr_fsns\fsns_attention_model",target=/models/fsns_attention_model -e MODEL_NAME=fsns_attention_model -t tensorflow/serving`

-> binds/mounts the local model path to the `/models/` path on the container

4. make model inferences over the container prediction API
issues with curl request: [here](https://stackoverflow.com/questions/52208668/tensorflow-serving-error-invalid-argument-json-object-does-not-have-named-inp)

4.1 get status of container: 

`curl http://localhost:8501/v1/models/fsns_attention_model` --> state should be set to `available`

4.2 get metadata of served model (e.g. input/output tensor): 

`curl http://localhost:8501/v1/models/fsns_attention_model/metadata`


# Relevant Literature

- review paper 


