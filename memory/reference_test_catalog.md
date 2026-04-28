---
name: Test file catalog
description: 23 test files with coverage descriptions and run commands; extracted from testing.md
type: reference
originSessionId: 836cb3d4-b894-4748-bd6d-60c75ed2f1ad
---
# Test File Catalog

Complete list of automated test files in `test_codes/` and `static/`.

| File | Coverage | Run Command |
|------|----------|-------------|
| `test_codes/test_project_crud_and_soft_delete.py` | Project CRUD, libraries, soft-delete, weather pagination/conflict (53 tests) | `python test_codes/test_project_crud_and_soft_delete.py` |
| `test_codes/test_weather_data.py` | Weather ingestion, seed data, file validation (13+6 tests) | `python test_codes/test_weather_data.py [--server]` |
| `test_codes/test_api_response.py` | Basic API response checks | `python test_codes/test_api_response.py` |
| `test_codes/test_weather_api.py` | Weather API endpoints | `python test_codes/test_weather_api.py` |
| `test_codes/test_fault_handle.py` | Fault handling | `python test_codes/test_fault_handle.py` |
| `test_codes/test_gas_and_pricing.py` | Gas consumption, heating/cooling season, TOU electricity, gas/water prices, regions (31 tests) | `python test_codes/test_gas_and_pricing.py` |
| `test_codes/test_comprehensive_api.py` | Login, dashboard, cooling module, faults, permissions, settings, systems CRUD, agriculture, water-fertilizer, data export, electricity prices, map/carbon (46 tests) | `python test_codes/test_comprehensive_api.py` |
| `test_codes/test_device_icons.py` | DeviceIconLibrary static analysis (G1), API device type validation (G2), page API contracts (G3); total 10 tests; G2/G3 require `--server` flag | `python test_codes/test_device_icons.py [--server]` |
| `static/test_device_icons.html` | Browser-side JS unit tests for DeviceIconLibrary.getIconSvg(): T1-T8, 22 assertions | Open `http://127.0.0.1:5000/static/test_device_icons.html` |
| `test_codes/test_control_logging.py` | Control log persistence, Chinese action names, fault logging, permission filter, DB schema (L1-L6, 16 tests) | `python test_codes/test_control_logging.py` |
| `test_codes/test_device_management.py` | Paginated device logs (DM1, 6), permission filtering (DM2, 3), add-device-from-library (DM3, 5); 14 tests | `python test_codes/test_device_management.py` |
| `test_codes/test_meter_reading.py` | Meter reading: first/consecutive readings, type isolation, legacy compat, recalculation, multi-project, soft delete (MR1-MR9, 25 tests) | `python test_codes/test_meter_reading.py` |
| `test_codes/test_device_library_expanded.py` | Expanded device library: template count, sensor+controller types, default_params, cumulative flag, industry filter (DL1-DL8, 69 tests) | `python test_codes/test_device_library_expanded.py` |
| `test_codes/test_permissions_rbac.py` | RBAC: login auth (P1, 7), user CRUD (P2, 10), admin protection (P3, 3), device control (P4, 8), log filtering (P5, 6), permission config (P6, 8), settings (P7, 5), security (P8, 5), edge cases (P9, 6); 58 tests | `python test_codes/test_permissions_rbac.py` |
| `test_codes/test_todo_verification.py` | Pending TODO: meteorology chart (G1), control log pagination (G2), meter type isolation (G3), price CRUD (G4), log boundaries (G5), cooling filter (G6); 38 tests | `python test_codes/test_todo_verification.py` |
| `test_codes/test_system_audit.py` | System audit: classification fix (G1), data dedup (G2), meter smoke (G3); 8 tests | `python test_codes/test_system_audit.py` |
| `test_codes/test_visual_svg_cards.py` | SVG card rendering: API validation (V1-V6, 5) + Vision MCP (6); 11 tests | `python test_codes/test_visual_svg_cards.py` |
| `test_codes/test_device_query_dedup.py` | Device query dedup: parameter API (G1, 6), device list (G2, 5), DB integrity (G3, 5), frontend contract (G4, 5), query/chart/export (G5, 5); 26 tests | `python test_codes/test_device_query_dedup.py` |
| `test_codes/test_schematic_layout.py` | Schematic layout CRUD (G1-G7, 30 tests): create/read/update/delete, duplicate, template, soft delete, edge cases, large data | `python test_codes/test_schematic_layout.py` |
| `test_codes/test_document_management.py` | Document management: config (G1, 4), system types (G2, 4), CRUD (G3, 6), permissions (G4, 4), cleanup (G5, 2); 20 tests | `python test_codes/test_document_management.py` |
| `test_codes/test_company_id_migration.py` | company_id migration: table structure (G1, 6), columns (G2, 8), filtering (G3, 5), login (G4, 3), API compat (G5, 3); 25 tests | `python test_codes/test_company_id_migration.py` |
| `test_codes/test_multi_tenant_permissions.py` | Multi-tenant RBAC: password hashing (G1, 4), login response (G2, 5), project filtering (G3, 6), endpoint auth (G4, 8), user CRUD (G5, 6), backward compat (G6, 4), device routing (G7, 3), menu filter (G8, 6); 42 tests | `NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe test_codes/test_multi_tenant_permissions.py` |
| `test_codes/test_device_query_trend.py` | Device query trend: company-project-device chain (G1), params (G2), data query (G3), trend chart (G4), Excel export (G5), device types (G6), comparison (G7), missing device (G8); ~46 tests | `NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe test_codes/test_device_query_trend.py` |

## ChangeLogs

- [2026-04-28] Initial: extracted from testing.md (21 entries) + added 2 missing (test_multi_tenant_permissions.py, test_device_query_trend.py); total 23 entries
