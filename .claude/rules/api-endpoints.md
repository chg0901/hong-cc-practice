# API Endpoints Reference

## Authentication
- `POST /api/login` - User login
- `POST /api/logout` - Clear session

## Projects & Devices
- `GET /api/projects` - List projects (?industry=&company= filters)
- `POST /api/projects` - Create project
- `GET /api/projects/<pid>` - Get project details (all fields)
- `PUT /api/projects/<pid>` - Update project
- `DELETE /api/projects/<pid>` - Soft delete project (status=0)
- `GET /api/projects/deleted` - List deleted projects
- `POST /api/projects/<pid>/restore` - Restore deleted project
- `POST /api/projects/<pid>/copy` - Copy project with systems and devices
- `GET /api/projects/<pid>/devices` - Get project devices
- `POST /api/projects/<pid>/devices` - Add device to project
- `DELETE /api/devices/<did>` - Soft delete device (status=0)

## Agriculture
- `GET /api/agriculture/meteorology/<pid>` - Weather data
- `GET /api/agriculture/soil/<pid>` - Soil moisture data

## Cooling/Heating
- `GET /api/cooling/overview/<pid>` - System overview
- `GET /api/cooling/energy/<pid>` - Energy data
- `GET /api/cooling/systems` - List cooling systems
- `GET /api/cooling/endpoints` - Get endpoints
- `GET /api/cooling/faults` - Get faults (client-side pagination)
- `GET /api/cooling/environment/<pid>` - Environment data (?hours=, ?start_date=, ?end_date=)

## Weather API (QWeather)
- `GET /api/weather/now` - Current weather
- `GET /api/weather/24h` - 24-hour forecast
- `GET /api/weather/72h` - 72-hour forecast (from 168h API, first 72 entries)
- `GET /api/weather/7d` - 7-day forecast
- `GET /api/weather/30d` - 30-day forecast (simulated)
- `GET /api/weather/sensor/latest` - Latest local sensor data (?project_id=)

## Device Control
- `POST /api/device/control/<did>` - Remote control (requires can_control=1)
- `GET /api/device/logs` - Audit log (?username=, ?page=, ?page_size=; returns `pagination` block)

## Data Management
- `POST /api/data/export` - Export historical data
- `POST /api/data/compare` - Compare data across periods

## System Library & Device Library
- `GET/POST /api/system-library` - List/Create system templates (?industry=)
- `PUT/DELETE /api/system-library/<sid>` - Update/Soft-delete system template
- `GET/POST /api/device-library` - List/Create device templates (?industry=&type=)
- `PUT/DELETE /api/device-library/<did>` - Update/Soft-delete device template

## Schematic Layout (SCADA Configuration Editor)
- `GET /api/schematic-layout` - List layouts (?project_id=&is_template=&industry=)
- `POST /api/schematic-layout` - Create layout (project_id, name, industry, layout_data, is_template, template_category)
- `PUT /api/schematic-layout/<lid>` - Update layout (name, layout_data, is_template, thumbnail)
- `DELETE /api/schematic-layout/<lid>` - Soft-delete layout
- `POST /api/schematic-layout/<lid>/duplicate` - Duplicate layout (target_project_id, name)
- `GET /api/schematic-layout/templates` - List template layouts (?industry=)

## Gas Consumption (Manual Entry / Meter Reading)
- `GET/POST /api/gas-consumption` - List/Add records (?project_id=&start_date=&end_date=&meter_type=)
- `PUT/DELETE /api/gas-consumption/<gid>` - Update/Soft-delete record
- Also available as `/api/meter-readings` (alias)

## Heating/Cooling Season
- `GET/POST /api/heating-cooling-season` - List/Add (?season_type=&company=)
- `PUT/DELETE /api/heating-cooling-season/<sid>` - Update/Soft-delete

## TOU Electricity Pricing
- `GET/POST /api/tou-electricity-prices` - List/Add (?project_id=&company=; includes hourly_schedule JSON)
- `PUT/DELETE /api/tou-electricity-prices/<pid>` - Update/Soft-delete

## Gas Pricing
- `GET/POST /api/gas-prices` - List/Add (?project_id=&company=)
- `PUT/DELETE /api/gas-prices/<gid>` - Update/Soft-delete

## Water Pricing
- `GET/POST /api/water-prices` - List/Add (?project_id=&company=)
- `PUT/DELETE /api/water-prices/<wid>` - Update/Soft-delete

## Other
- `GET /api/regions` - Province/city/district address data
- `GET/POST /api/system/settings` - Get/Update system settings

## Document Management (项目资料管理)
- `GET /api/documents/config` - Get storage config (upload_base, max_file_size)
- `GET /api/documents/system-types/<project_id>` - Get available system folder types
- `GET /api/documents/list/<project_id>` - List documents (?type= filter)
- `POST /api/documents/upload` - Upload document (form: project_id, doc_type, description, system_type, file)
- `POST /api/documents/rename` - Rename document (json: doc_id, new_name)
- `DELETE /api/documents/delete/<doc_id>` - Delete document (admin only)
- `GET /api/documents/view/<doc_id>` - View/preview document

## Device Query (系统运行监控 - 设备级数据查询)
- `GET /api/device-query/companies` - Company list (multi-tenant filtered)
- `GET /api/device-query/projects` - Project list (?company= filter)
- `GET/POST /api/device-query/devices` - Device list with system classification
- `GET /api/device-query/params/<id>` - Device parameters
- `GET /api/device-query/config-params/<id>` - Configurable parameters
- `POST /api/device-query/config-params/<id>` - Update configurable parameters
- `POST /api/device-query/config-params/<id>/reset` - Reset to defaults
- `POST /api/device-query/query` - Data query with pagination
- `POST /api/device-query/chart-data` - Trend chart data (?aggregate=raw/5min/hour/day)
- `POST /api/device-query/export` - Generate Excel export
- `GET /api/device-query/download` - Download exported file (?filepath=)
- `GET /api/device-query/device-types` - Available device types
- `GET /api/device-query/type-params/<type>` - Parameters for a device type
- `POST /api/device-query/compare` - Cross-device parameter comparison
