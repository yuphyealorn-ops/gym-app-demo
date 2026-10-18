# FORM — gym companion

A local gym app with body check-ins, a camera/photo pose scan, progress history, and a workout log. The browser runs a pretrained pose model; body-composition estimates use measurements that you enter. No account, Python packages, Node dependencies, or training run are required.

## Run

Requires Python 3.10 or newer and a modern browser with WebAssembly support.

```sh
cd gym-app
python3 app.py
```

Open **http://localhost:8000**. You can also use `python3 app.py --open` or choose another port with `python3 app.py --port 8080`. The launcher works from any directory when given its full path. It binds to your own computer (`127.0.0.1`) and serves only `public/`.

## Use

1. Enter your age, height in centimetres, weight in kilograms, tape-measured waist, and equation sex.
2. Use the camera or upload a JPG, PNG, or WebP photo to check full-body framing and see the detected pose. Include your head and feet, face the camera, and leave space around your body. Camera access needs the browser's permission on localhost; use the launcher instead of opening `index.html` directly.
3. Save check-ins to follow your measurements over time. Log workouts and export your saved data from the app.

The photo scan locates body landmarks. It does not measure body fat, muscle, or waist circumference. RFM estimates are calculated separately from the entered measurements, with ages 20–69 supported by this app. Fat-free mass includes more than muscle. All metrics are experimental wellness estimates, with no claim of clinical accuracy. See the [model card](docs/model-card.md) for the equation, evidence, and limitations.

## Models and offline use

The first scan loads the pinned MediaPipe Tasks Vision **1.1.1** runtime and **Pose Landmarker Lite, float16, revision 1**. This needs internet access unless the six assets are installed locally. Measurements and workout logging work without downloading a model.

To download the assets once for subsequent local use:

```sh
python3 scripts/setup_models.py
```

Reload the app after setup finishes. The script uses verified HTTPS, timeouts, and atomic file replacement; interrupted transfers do not publish partial files. Non-empty existing files are reused. If an asset is corrupt, redownload with `python3 scripts/setup_models.py --force`.

`public/models/assets-manifest.json` records source URLs, sizes, and SHA-256 hashes. These hashes describe the retrieved files; they are **not** verification against independent publisher checksums. `/api/model-status` reports local availability only when all six files exist and are non-empty. It does not prove that inference succeeds. Downloaded assets are excluded from version control.

When local assets are unavailable, the browser contacts `cdn.jsdelivr.net` and `storage.googleapis.com` for runtime/model files. Those services receive normal network request metadata. Photo pixels are processed in the browser; this app has no photo-upload endpoint. Saved measurements and workouts use browser local storage, which is not encrypted and is separate for each browser/origin. Changing ports changes the storage origin. Use a personal browser profile and export records before clearing browser data.

## How this relates to the reference project

[`../term project/src/train_cnn.py`](../term%20project/src/train_cnn.py) compares a custom CNN with pretrained ResNet18 for five image classes. This app carries over that decision process and the separation between inference, data, and presentation. Its different target is pose estimation, so MediaPipe provides a suitable pretrained model without inventing a training dataset.

[`../bmi.csv`](../bmi.csv) has `Gender`, `Height`, `Weight`, and `Index`; it has no photographs, waist measurements, or measured body-fat labels. The reference image classifier also has no body-composition targets. Neither dataset can support a defensible body-fat-from-photo training run. No new model was trained and no benchmark score is claimed. This gym project is an adaptation, **not fulfillment of the original car/truck/airplane/ship/horse classification assignment**.

The [explanation notebook](Gym_App_Explained.ipynb) runs with Python's standard library and demonstrates the local dataset audit, formula, and worked example. [Project notes](docs/project-notes.md) describe the design and what a future training study would require.

## Verify

Python verifies the read-only server, path confinement, security headers, asset availability, and atomic downloader. Node.js is only needed to run the JavaScript tests:

```sh
python3 -m unittest discover -s tests -p 'test_*.py'
node --test tests/*.test.js
```

The server tests exercise HTTP parsing in memory, so they also work where opening local sockets is prohibited. Download tests use controlled responses, not network downloads. Automated formula/storage tests and server tests do not establish accuracy on people. Live camera inference and complete upstream model downloads have not been verified in the restricted development environment.

## Files

| File | Purpose |
| --- | --- |
| `app.py` | Local Python server; `/api/health` and `/api/model-status` |
| `public/` | HTML, CSS, browser modules, and optional model assets |
| `public/js/scanner.js` | Camera/photo handling and pretrained pose inference |
| `public/js/metrics.js` | Measurement validation, composition equation, and framing checks |
| `public/js/storage.js` | Validated browser persistence and data export |
| `scripts/setup_models.py` | Optional pinned runtime/model downloader |
| `tests/` | JavaScript and Python behavior tests |
| `docs/model-card.md` | Model sources, licenses, intended use, and limitations |
| `Gym_App_Explained.ipynb` | Executable walkthrough of the ML decision and calculation |
