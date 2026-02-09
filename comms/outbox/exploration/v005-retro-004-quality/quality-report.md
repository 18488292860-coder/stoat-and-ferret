# Full Quality Gate Output

## Ruff Lint Check

```
$ uv run ruff check src/ tests/

All checks passed!
```

## Ruff Format Check

```
$ uv run ruff format --check src/ tests/

103 files already formatted
```

## Mypy Type Check

```
$ uv run mypy src/

Success: no issues found in 44 source files
```

## Pytest

```
$ uv run pytest tests/ --tb=short

============================= test session starts =============================
platform win32 -- Python 3.13.11, pytest-9.0.2, pluggy-1.6.0
rootdir: C:\Users\grant\Documents\projects\stoat-and-ferret
configfile: pyproject.toml
plugins: anyio-4.12.1, hypothesis-6.151.5, asyncio-1.3.0, cov-7.0.0
asyncio: mode=Mode.AUTO, debug=False
collected 642 items

tests\examples\test_property_example.py ....                             [  0%]
tests\test_api\test_app.py .......                                       [  1%]
tests\test_api\test_clips.py .............                               [  3%]
tests\test_api\test_di_wiring.py .....                                   [  4%]
tests\test_api\test_factory_api.py .....                                 [  5%]
tests\test_api\test_gui_static.py .....                                  [  6%]
tests\test_api\test_health.py ......                                     [  7%]
tests\test_api\test_jobs.py .....                                        [  7%]
tests\test_api\test_middleware.py .........                              [  9%]
tests\test_api\test_projects.py ...........                              [ 10%]
tests\test_api\test_settings.py .................                        [ 13%]
tests\test_api\test_thumbnail_endpoint.py .....                          [ 14%]
tests\test_api\test_videos.py ............................               [ 18%]
tests\test_async_repository_contract.py ................................ [ 23%]
..................                                                       [ 26%]
tests\test_audit_logging.py .............                                [ 28%]
tests\test_blackbox\test_core_workflow.py .......                        [ 29%]
tests\test_blackbox\test_edge_cases.py ...........                       [ 31%]
tests\test_blackbox\test_error_handling.py ............                  [ 33%]
tests\test_clip_model.py .........                                       [ 34%]
tests\test_clip_repository_contract.py ....................              [ 37%]
tests\test_contract\test_ffmpeg_contract.py .....ssssss..........        [ 40%]
tests\test_contract\test_repository_parity.py ........                   [ 42%]
tests\test_contract\test_search_parity.py .......                        [ 43%]
tests\test_coverage\test_import_fallback.py ...                          [ 43%]
tests\test_db_schema.py ........                                         [ 45%]
tests\test_doubles\test_inmemory_isolation.py ..........                 [ 46%]
tests\test_doubles\test_inmemory_job_queue.py ...........                [ 48%]
tests\test_doubles\test_seed_helpers.py ........                         [ 49%]
tests\test_executor.py .....................sss....                      [ 53%]
tests\test_factories.py .............                                    [ 55%]
tests\test_ffprobe.py .....sss..ss..                                     [ 58%]
tests\test_integration.py .............                                  [ 60%]
tests\test_jobs\test_asyncio_queue.py ............                       [ 61%]
tests\test_jobs\test_worker.py ....                                      [ 62%]
tests\test_logging.py ....                                               [ 63%]
tests\test_observable.py ..................                              [ 66%]
tests\test_project_repository_contract.py ........................       [ 69%]
tests\test_pyo3_bindings.py ............................................ [ 76%]
.............................                                            [ 81%]
tests\test_repository_contract.py ...................................... [ 87%]
.........                                                                [ 88%]
tests\test_security\test_input_sanitization.py .......................   [ 92%]
tests\test_security\test_path_validation.py .......s.....                [ 94%]
tests\test_smoke.py ....                                                 [ 94%]
tests\test_thumbnail_service.py ...........                              [ 96%]
tests\test_websocket.py .............                                    [ 98%]
tests\test_ws_endpoint.py ..........                                     [100%]

=============================== warnings summary ===============================
tests/test_blackbox/test_core_workflow.py::TestProjectLifecycle::test_create_and_retrieve_project
  ResourceWarning: unclosed database in <sqlite3.Connection object>
  (12 instances of this warning)

================ 627 passed, 15 skipped, 12 warnings in 10.18s ================
```

## Coverage Report

```
Name                                             Stmts   Miss Branch BrPart  Cover   Missing
--------------------------------------------------------------------------------------------
src\stoat_ferret\__init__.py                         2      0      0      0   100%
src\stoat_ferret\api\__init__.py                     0      0      0      0   100%
src\stoat_ferret\api\__main__.py                    10     10      2      0     0%   6-32
src\stoat_ferret\api\app.py                         76     16     14      1    81%   61-87
src\stoat_ferret\api\middleware\__init__.py          0      0      0      0   100%
src\stoat_ferret\api\middleware\correlation.py      18      0      0      0   100%
src\stoat_ferret\api\middleware\metrics.py          19      0      0      0   100%
src\stoat_ferret\api\routers\__init__.py             0      0      0      0   100%
src\stoat_ferret\api\routers\health.py              50      3      8      0    95%   78-80
src\stoat_ferret\api\routers\jobs.py                12      0      0      0   100%
src\stoat_ferret\api\routers\projects.py           106      6     30      5    92%   47, 65, 83-85, 332, 340
src\stoat_ferret\api\routers\videos.py              65      1     16      1    98%   49
src\stoat_ferret\api\routers\ws.py                  23      1      0      0    96%   27
src\stoat_ferret\api\schemas\__init__.py             0      0      0      0   100%
src\stoat_ferret\api\schemas\clip.py                25      0      0      0   100%
src\stoat_ferret\api\schemas\job.py                 11      0      0      0   100%
src\stoat_ferret\api\schemas\project.py             20      0      0      0   100%
src\stoat_ferret\api\schemas\video.py               40      0      0      0   100%
src\stoat_ferret\api\services\__init__.py            0      0      0      0   100%
src\stoat_ferret\api\services\scan.py               63      2     16      2    95%   119, 143
src\stoat_ferret\api\services\thumbnail.py          29      0      4      0   100%
src\stoat_ferret\api\settings.py                    23      0      0      0   100%
src\stoat_ferret\api\websocket\__init__.py           0      0      0      0   100%
src\stoat_ferret\api\websocket\events.py            14      0      0      0   100%
src\stoat_ferret\api\websocket\manager.py           33      0      6      0   100%
src\stoat_ferret\db\__init__.py                      8      0      0      0   100%
src\stoat_ferret\db\async_repository.py            125      9     18      1    93%   38, 49, 60, 72, 84, 98, 106, 117, 170
src\stoat_ferret\db\audit.py                        18      0      0      0   100%
src\stoat_ferret\db\clip_repository.py              75      5     10      0    94%   34, 45, 56, 70, 81
src\stoat_ferret\db\models.py                       87      0      2      0   100%
src\stoat_ferret\db\project_repository.py           75      5     10      0    94%   36, 47, 59, 73, 84
src\stoat_ferret\db\repository.py                  120      7     22      0    95%   35, 46, 57, 69, 81, 95, 106
src\stoat_ferret\db\schema.py                       34      0      0      0   100%
src\stoat_ferret\ffmpeg\__init__.py                  6      0      0      0   100%
src\stoat_ferret\ffmpeg\executor.py                 60      6      8      0    91%   58, 93-105
src\stoat_ferret\ffmpeg\integration.py              20      4      0      0    80%   76-83
src\stoat_ferret\ffmpeg\metrics.py                   4      0      0      0   100%
src\stoat_ferret\ffmpeg\observable.py               27      0      0      0   100%
src\stoat_ferret\ffmpeg\probe.py                    51     18      6      0    61%   82-94, 110-128
src\stoat_ferret\jobs\__init__.py                    2      0      0      0   100%
src\stoat_ferret\jobs\queue.py                     156      7     24      2    95%   69, 83, 97, 196-197, 219->227, 380-381
src\stoat_ferret\logging.py                         16      0      2      0   100%
src\stoat_ferret_core\__init__.py                   36      0      0      0   100%
--------------------------------------------------------------------------------------------
TOTAL                                             1559    100    198     12    93%
Required test coverage of 80.0% reached. Total coverage: 93.28%
```

## Summary

All four quality gates pass:
- **ruff check**: 0 lint violations
- **ruff format**: 103 files already correctly formatted
- **mypy**: 0 type errors across 44 source files
- **pytest**: 627 passed, 15 skipped, 93.28% coverage (threshold: 80%)

No fixes were required. No additional rounds were needed.
