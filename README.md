# modular-filter-examples

Examples for creating modular components that filter data capture. This repo contains code for:
* A modular camera that filters camera data capture, in both go (`colorfilter` directory) and python (`pycolorfilter` directory). Configuring data capture on this component only keeps the data captured on the underlying camera if the vision service detects a certain color in the captured image.
* A modular sensor that that filters ultrasonic sensor capture, in go (`sensorfilter` directory). Configuring data capture on this component only keeps data captured on the underlying sensor if there is a significant change in distance readings.

## Usage

### Build binary
* Navigate to example's module subdirectory (ex. for go color filter, `cd colorfilter/module`)
* Run `go build`

### Add to robot config

Color filter example:
```
{
  "modules": [
    {
      "executable_path": <path_to_binary_or_script>,
      "type": "local",
      "name": "colorfilter"
    }
  ],
  "components": [
    {
      "name": "actualcam",
      "type": "camera"
      ...
    },
    {
      "name": "colorfiltercam",
      "type": "camera",
      "model": "example:camera:colorfilter",
      "attributes": {
        "actual_cam": "actualcam",
        "vision_service": "my_color_detector"
      },
      "depends_on": [
        "actualcam",
        "my_color_detector"
      ],
      "service_configs": [
        {
          "type": "data_manager",
          "attributes": {
            "capture_methods": [
              {
                "method": "ReadImage",
                "additional_params": {
                  "mime_type": "image/jpeg"
                },
                "capture_frequency_hz": 1
              }
            ]
          }
        }
      ]
    }
  ],
  "services": [
    {
      "name": "dm",
      "type": "data_manager",
      "attributes": {
        "sync_interval_mins": 0.1,
        "capture_dir": "",
        "tags": [],
        "additional_sync_paths": []
      }
    },
    {
      "type": "vision",
      "attributes": {
        "hue_tolerance_pct": 0.03,
        "segment_size_px": 100,
        "detect_color": "#C6907A"
      },
      "model": "color_detector",
      "name": "my_color_detector"
    }
  ]
}
```

Sensor filter example:
```
{
  "modules": [
    {
      "executable_path": <path_to_binary>,
      "type": "local",
      "name": "sensorfilter"
    }
  ]
  "components": [
    {
      "name": "actual_sensor",
      "type": "sensor",
      "model": "ultrasonic",
      "attributes": { ... }
    },
    {
      "name": "filter_sensor",
      "type": "sensor",
      "model": "example:sensor:sensorfilter",
      "service_configs": [
        {
          "type": "data_manager",
          "attributes": {
            "capture_methods": [
              {
                "additional_params": {},
                "capture_frequency_hz": 1,
                "method": "Readings"
              }
            ]
          }
        }
      ],
      "attributes": {
        "actual_sensor": "actual_sensor"
      },
      "depends_on": [
        "actual_sensor"
      ]
    }
  ],
  "services": [
    {
      "type": "data_manager",
      "attributes": {
        "tags": [],
        "additional_sync_paths": [],
        "sync_interval_mins": 0.1,
        "capture_dir": ""
      },
      "name": "dm"
    }
  ]
}
```
