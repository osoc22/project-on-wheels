# Import process

This document aims to shed some light on the import process.

The process currently consists of the following steps:

1. Convert the original data to a GeoJSON format with tags that are compatible with OSM.
2. Upload the GeoJSON to a MapRoulette challenge.
3. Add challenge layer to MapComplete theme.

## Converting to GeoJSON

This step isn't fully defined for every dataset,
but for the OnWheels data, there are some conversion scripts in the [MapComplete repo](https://github.com/pietervdvn/MapComplete/blob/feature/maproulette/scripts/onwheels).

For converting you can run `ts-node convert-data.ts <input file> <output file>` (e.g `ts-node convert-data.ts data/onwheels.csv data/onwheels`).

The most important thing is that the following properties are present in the GeoJSON:

* `id`: A unique identifier for the feature.
* `name`: The name of item to import.
* `tags`: A list of tags that should be applied to the item in OSM. (eg. `tourism=hotel;name=My fancy hotel;...`)

## Uploading to MapRoulette

To manage the items we're using MapRoulette.

1. First login to [MapRoulette](https://maproulette.org/).

![Create & Manage](img/create-manage.png)

2. Go to the [Create & Manage](https://maproulette.org/admin/projects) page.
3. Select a project, or create a new one.
4. Once you've selected a project, you can add a new challenge.
5. Fill in all fields, most are well documented on the page itself, for the 'Location of your task data' select 'I want to upload a GeoJSON file'.
6. Upload the GeoJSON file we created earlier.
7. Once you're at the challenge page we can find the ID of the challenge, we'll need it later. It's located in the URL.

![URL](img/URL.png)

## Adding a challenge layer to MapComplete

1. Find the theme you want to add the challenge to.
2. Add it to the layers, by reusing the `maproulette_challenge` layer.

```json
{
    "builtin": "maproulette_challenge",
    "override": {
        "source": {
            "geoJson": "https://maproulette.org/api/v2/challenge/view/XXXX"
        },
        "calculatedTags": [
            "_closest_osm_YYYY=feat.closest('YYYY')?.properties?.id",
            "_closest_osm_YYYY_distance=feat.distanceTo(feat.properties._closest_osm_YYYY)",
            "_has_closeby_YYYY=Number(feat.properties._closest_osm_YYYY_distance) < 50 ? 'yes' : 'no'"
        ],
        "+tagRenderings": [
            {
                "id": "import-button",
                "condition": "_has_closeby_YYYY=no",
                "render": {
                    "special": {
                    "type": "import_button",
                    "targetLayer": "YYYY",
                    "tags": "tags",
                    "text": {
                        "en": "Import"
                    },
                    "icon": "./assets/svg/addSmall.svg",
                    "location_picker": "photo",
                    "maproulette_id": "mr_taskId"
                    }
                }
            },
            {
                "id": "tag-apply-button",
                "condition": "_has_closeby_YYYY=yes",
                "render": {
                    "special": {
                    "type": "tag_apply",
                    "tags_to_apply": "$tags",
                    "message": {
                        "en": "Add all the suggested tags"
                    },
                    "image": "./assets/svg/addSmall.svg",
                    "id_of_object_to_apply_this_one": "_closest_osm_YYYY"
                    }
                }
            }
        ]
    }
}
```

In this example `YYYY` will need to be replaced by the name of the MapComplete layer and `XXXX` by the challenge ID from the MapRoulette challenge.
