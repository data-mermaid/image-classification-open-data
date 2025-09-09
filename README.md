# Community coral reef image classification training data

## Summary

The `coral-reef-training` AWS S3 bucket provides a single, open, well-structured, growing, community-sourced repository of coral reef image classification training data. Hosted at **s3://coral-reef-training**, this bucket supports global efforts in coral reef conservation through standardized, machine-learning-ready imagery and annotations.

The bucket serves as the image storage backend for MERMAID’s image classification workflows and to distribute confirmed and scrubbed MERMAID coral reef image data, but it also provides a shared location where partners including [CoralNet](https://coralnet.ucsd.edu/) can contribute to and benefit from collective ML model development, each according to its own data structures and policies. Data in the bucket is free and open for public access; only contributing organizations have write access to their own data prefixes.

By centralizing and standardizing coral reef image data, this initiative accelerates collaboration across scientific, conservation, and machine learning communities and facilitates the creation of a common, evolving image classification model for coral reefs worldwide.

## Data

All community coral reef image classification training data are available in a single AWS bucket at **s3://coral-reef-training** in the `us-east-1` region, enabling streamlined access. Each partner's data is stored with a separate prefix (i.e. in a separate directory), and is organized according to the standards of that organization; partner-specific data descriptions are below. Each partner's labels are maintained independently; MERMAID provides an attempt to crosswalk labels with its [LabelMapping API endpoint](https://api.datamermaid.org/v1/classification/labelmappings/) (see usage example).

The bucket is entirely free and open for reading; each partner organization has write access only to the files corresponding to its prefix.  


### Data access

#### Console and CLI

The **s3://coral-reef-training** bucket can be accessed via the AWS web console using this url:   
https://s3.console.aws.amazon.com/s3/buckets/coral-reef-training?region=us-east-1&bucketType=general&tab=objects

S3 objects can be downloaded via the AWS CLI, which can be installed according to [AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

For example, to download the aggregate MERMAID annotations described below:
```shell
aws s3 cp s3://coral-reef-training/mermaid/mermaid_confirmed_annotations.parquet ./
```

#### Programmatic access

In Python, [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)'s [S3 service](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html) is the established way to list and read files via the S3 API. [s3fs](https://s3fs.readthedocs.io/en/latest/) is great for reading parquet and csv files directly.

For direct loading of data, [pandas](https://pandas.pydata.org/) and [pyarrow](https://arrow.apache.org/docs/python/index.html
) are a good option:

```python
import pandas as pd

df = pd.read_parquet(
    's3://coral-reef-training/mermaid/mermaid_confirmed_annotations.parquet',
    storage_options={'anon': True}
)
```

In R, the [arrow](https://arrow.apache.org/docs/r/) package can be used to read the parquet as follows:

```r
library(arrow)

df_annotations <- arrow::read_parquet(
  "https://coral-reef-training.s3.us-east-1.amazonaws.com/mermaid/mermaid_confirmed_annotations.parquet"
)
```

### MERMAID

Prefix: `mermaid/`  
Update cadence:
- image-specific data: 
  - an image and its corresponding image thumbnail are created immediately upon upload
  - after inference is run successfully by the MERMAID classifier, a featurevector file is created
  - upon successful validation and submission of the benthic photo quadrat transect containing the image, a csv of the annotations for that image is created
- aggregate: once per day `mermaid_confirmed_annotations.parquet`, containing all confirmed annotations for non-test-project submitted data in MERMAID, is updated

#### per-image 

- `<image_id>.png`  
The image stored by MERMAID is a version of the one uploaded by the MERMAID user:
  - saved (losslessly) in png format (dimensions unaltered)
  - [EXIF](https://en.wikipedia.org/wiki/Exif) data stripped, including location and timestamp
  - filename is the `<image_id>` assigned by the MERMAID API; if authenticated and a project member, one can access more information using this id (see usage example)
- `<image_id>_thumbnail.png`  
same image shrunk to fit within a 500x500 extent (maintaining aspect ratio) for display purposes
- `<image_id>_featurevector`  
file corresponding to a set of point locations on a single image -- see [pyspacer](https://github.com/coralnet/pyspacer) for more information; may not be present for images processed with future classifiers
- `<image_id>_annotations.csv`  
confirmed annotations for an image (ordered by `row`, `col`), with fields:

| field name               | type     | description                                                                                                          |
|--------------------------|----------|----------------------------------------------------------------------------------------------------------------------|
| `id`                     | uuid     | unique annotation id                                                                                                 |
| `image_id`               | uuid     | unique id assigned to image by MERMAID, corresponding to image filename                                              |
| `point_id`               | uuid     | id of point unique across all of MERMAID (not just image)                                                            |
| `row`                    | integer  | number of pixels from top of image for annotation point                                                              |
| `col`                    | integer  | number of pixels from left of image for annotation point                                                             |
| `benthic_attribute_id`   | uuid     | unique id of MERMAID benthic attribute                                                                               |
| `benthic_attribute_name` | string   | name of MERMAID benthic attribute corresponding to `benthic_attribute_id`                                            |
| `growth_form_id`         | uuid     | unique id of MERMAID benthic growth form; may be null                                                                |
| `growth_form_name`       | string   | name of MERMAID benthic growth form corresponding to `growth_form_id`; may be null                                   |
| `updated_on`             | datetime | timestamp of most recent annotation update, expressed in [ISO 8601 standard](https://en.wikipedia.org/wiki/ISO_8601) |

#### aggregate

Once per day `s3://coral-reef-training/mermaid/mermaid_confirmed_annotations.parquet` is updated. This file contains all confirmed annotations for non-test-project submitted [Benthic Photo Quadrat transects](https://datamermaid.org/documentation/benthic-photo-quadrat) in MERMAID, and is ordered by (`image_id`, `row`, `col`) with fields:  

| field name               | type     | description                                                                                                                                                                                                                                                                                                                         |
|--------------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`                     | uuid     | unique annotation id                                                                                                                                                                                                                                                                                                                |
| `image_id`               | uuid     | unique id assigned to image by MERMAID, corresponding to image filename                                                                                                                                                                                                                                                             |
| `point_id`               | uuid     | id of point unique across all of MERMAID (not just image)                                                                                                                                                                                                                                                                           |
| `row`                    | integer  | number of pixels from top of image for annotation point                                                                                                                                                                                                                                                                             |
| `col`                    | integer  | number of pixels from left of image for annotation point                                                                                                                                                                                                                                                                            |
| `benthic_attribute_id`   | uuid     | unique id of MERMAID benthic attribute                                                                                                                                                                                                                                                                                              |
| `benthic_attribute_name` | string   | name of MERMAID benthic attribute corresponding to `benthic_attribute_id`                                                                                                                                                                                                                                                           |
| `growth_form_id`         | uuid     | unique id of MERMAID benthic growth form; may be null                                                                                                                                                                                                                                                                               |
| `growth_form_name`       | string   | name of MERMAID benthic growth form corresponding to `growth_form_id`; may be null                                                                                                                                                                                                                                                  |
| `updated_on`             | datetime | timestamp of most recent annotation update, expressed in [ISO 8601 standard](https://en.wikipedia.org/wiki/ISO_8601)                                                                                                                                                                                                                |
| `region_id`              | uuid     | id assigned to one of 12 MERMAID regions, taken from the [MEOW](https://www.worldwildlife.org/publications/marine-ecoregions-of-the-world-a-bioregionalization-of-coastal-and-shelf-areas) 'realms' (largest most aggregated polygons), in which the Benthic Photo Quadrat transect containing the annotation's image was collected |
| `region_name`            | string   | name of MERMAID region corresponding to `region_id`                                                                                                                                                                                                                                                                                 |

## Usage example

TODO
- fetch MERMAID annotations
- fetch Coralnet annotations
- label normalization
- iterate over subset of images and get image files and details
- indicate training classifier

## Tutorials

TODO

## License

[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

## Suggested attribution

TODO

## Change log

**2025-08-06** initial commit
