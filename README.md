# ziso-shuffle-apps

Shuffle apps maintained for the Ziso SOC platform, in the layout Shuffle loads
from a git repository (`<app>/<version>/` with `api.yaml`, `Dockerfile`,
`requirements.txt` and `src/app.py`).

## cortex 1.0.1

A port of the upstream `cortex` 1.0.0 app from
[Shuffle/python-apps](https://github.com/Shuffle/python-apps) (MIT) to the
current Shuffle app SDK. The actions and their parameters are unchanged:
`get_available_analyzers`, `run_available_analyzers`, `run_analyzer` and
`get_analyzer_result`.

Why it exists: the upstream 1.0.0 imports `walkoff_app_sdk.app_base`, which the
current `frikky/shuffle:app_sdk` base image no longer provides in a working
form (`ModuleNotFoundError: No module named 'walkoff_app_sdk.shuffle_sdk'`), so
the app crash-loops on start. Current Shuffle apps install `shuffle_sdk` from
PyPI instead.

Changes from upstream 1.0.0:

- imports `AppBase` from `shuffle_sdk`;
- installs `shuffle_sdk` with `--no-deps` and pins its runtime dependencies
  in `requirements.txt` to the versions the current Shuffle HTTP app ships
  (`shuffle_sdk` itself pins `requests==2.32.3` and `urllib3==2.3.0`, both with
  published CVEs);
- casts `tlp` to an integer and accepts a boolean `force`;
- accepts MISP and DFIR-IRIS observable types (`ip-dst`, `sha256`, `hostname`,
  `email-src`, ...) as `datatype` and maps them to Cortex data types, so an IRIS
  IOC type can be passed straight through;
- version 1.0.1, so its image tag (`frikky/shuffle:cortex_1.0.1`) does not
  collide with the 1.0.0 tag on Docker Hub, which a Shuffle swarm would pull
  over a locally built image.

## Load into Shuffle

In the Shuffle UI: Apps, Download from a Git repository, with this repository's
URL and branch `main`. Or through the API with an admin key:

```bash
curl -X POST -H "Authorization: Bearer $SHUFFLE_APIKEY" -H 'Content-Type: application/json' \
  -d '{"url":"https://github.com/lmakonem/ziso-shuffle-apps","branch":"main"}' \
  "$SHUFFLE_BACKEND/api/v1/apps/download_remote"
```

Shuffle builds the image on its own host. Authentication for the app takes the
Cortex URL and a per-user Cortex API key; never commit either.

## License

MIT, as upstream. See [LICENSE](LICENSE).
