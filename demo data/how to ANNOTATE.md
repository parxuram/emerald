### Demo Usage

1. Upload a traffic video to the system.
2. Configure scene regions such as lanes, stop lines, no-parking zones, and direction boundaries.
3. Run the detection pipeline.
4. The system automatically detects vehicles, tracks movement, identifies violations, performs OCR, and generates evidence records.
5. Review violations, analytics, and reports from the generated outputs.

### Scene Calibration

To configure custom road layouts, use **[CVAT AI](https://app.cvat.ai?utm_source=chatgpt.com)**:

1. Upload a frame from the target camera.
2. Draw polygons for lanes, stop lines, parking zones, and restricted areas.
3. Export or note the polygon coordinates.
4. Paste these coordinates into the model configuration file.
5. Run the system with the updated scene calibration settings.

This enables the same EMERALD framework to be adapted to different junctions and camera viewpoints with minimal effort.
