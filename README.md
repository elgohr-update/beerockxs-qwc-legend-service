[![](https://github.com/qwc-services/qwc-legend-service/workflows/build/badge.svg)](https://hub.docker.com/repository/docker/sourcepole/qwc-legend-service)
[![](https://img.shields.io/docker/pulls/sourcepole/qwc-legend-service)](https://hub.docker.com/repository/docker/sourcepole/qwc-legend-service)

QWC Legend Service v2
=====================

Acts as a proxy between the client and the OGC service for GetLegendGraphic request,
allowing to return custom legend graphics instead of the ones generated by the WMS server.

**v2** (WIP): add support for multitenancy and replace QWC Config service with static config and permission files.

**Note:** Requires a QGIS server running on `default_qgis_server_url`.


Configuration
-------------

The static config and permission files are stored as JSON files in `$CONFIG_PATH` with subdirectories for each tenant,
e.g. `$CONFIG_PATH/default/*.json`. The default tenant name is `default`.


### Data Service config

* [JSON schema](schemas/qwc-legend-service.json)
* File location: `$CONFIG_PATH/<tenant>/legendConfig.json`

Example:
```jsonc
{
  "$schema": "https://raw.githubusercontent.com/qwc-services/qwc-legend-service/v2/schemas/qwc-legend-service.json",
  "service": "legend",
  "config": {
    // QGIS Server URL
    "default_qgis_server_url": "http://localhost:8001/ows/",
    // base path to legend images
    "legend_images_path": "/PATH/TO/LEGENDS/"
  },
  "resources": {
    "wms_services": [
      {
        "name": "qwc_demo",
        "root_layer": {
          "name": "qwc_demo",
          "layers": [
            {
              "name": "edit_demo",
              "layers": [
                {
                  "name": "edit_points",
                  // using custom legend image
                  "legend_image": "edit_demo.png"
                },
                {
                  "name": "edit_lines",
                  // using custom legend image
                  "legend_image": "edit_lines.png"
                },
                {
                  // using WMS GetLegendGraphics by default
                  "name": "edit_polygons"
                }
              ]
            },
            {
              "name": "geographic_lines"
            },
            {
              "name": "country_names"
            },
            {
              "name": "states_provinces"
            },
            {
              "name": "countries"
            }
          ]
        }
      }
    ]
  }
}
```

**Note**: the `legend_image` path for custom legend graphics is relative to `legend_images_path` and may contain subdirectories


### Permissions

* [JSON schema](https://github.com/qwc-services/qwc-services-core/blob/master/schemas/qwc-services-permissions.json)
* File location: `$CONFIG_PATH/<tenant>/permissions.json`

Example:
```json
{
  "$schema": "https://raw.githubusercontent.com/qwc-services/qwc-services-core/master/schemas/qwc-services-permissions.json",
  "users": [
    {
      "name": "demo",
      "groups": ["demo"],
      "roles": []
    }
  ],
  "groups": [
    {
      "name": "demo",
      "roles": ["demo"]
    }
  ],
  "roles": [
    {
      "role": "public",
      "permissions": {
        "wms_services": [
          {
            "name": "qwc_demo",
            "layers": [
              {
                "name": "qwc_demo"
              },
              {
                "name": "edit_demo"
              },
              {
                "name": "edit_points"
              },
              {
                "name": "edit_lines"
              },
              {
                "name": "edit_polygons"
              },
              {
                "name": "geographic_lines"
              },
              {
                "name": "country_names"
              },
              {
                "name": "states_provinces"
              },
              {
                "name": "countries"
              }
            ]
          }
        ]
      }
    }
  ]
}
```


### Legend images

Place your custom legend images in `legend_images_path` and set the `legend_image` paths in the layer configurations accordingly.

**TODO:** The service accepts a `type` query-parameter to distinguish three types of legend graphics:

 - `type=thumbnail`: Image for the thumbnails in the QWC2 layer tree
 - `type=tooltip`: Image for the tooltips when hovering over the thumbnails in the QWC2 layer tree
 - `type=default`: Default image, i.e. in the legend print or in the layer info dialog


Usage
-----

Set the `CONFIG_PATH` environment variable to the path containing the service config and permission files when starting this service (default: `config`).

Set the `QWC2_PATH` environment variable to the path containing your QWC2 production build.


Base URL:

    http://localhost:5014/

API documentation:

    http://localhost:5014/api/

Sample request:

    http://localhost:5014/qwc_demo?SERVICE=WMS&REQUEST=GetLegendGraphic&LAYER=qwc_demo&format=image/png


Development
-----------

Create a virtual environment:

    virtualenv --python=/usr/bin/python3 --system-site-packages .venv

Without system packages:

    virtualenv --python=/usr/bin/python3 .venv

Activate virtual environment:

    source .venv/bin/activate

Install requirements:

    pip install -r requirements.txt
    pip install flask_cors

Start local service:

    CONFIG_PATH=/PATH/TO/CONFIGS/ python server.py
