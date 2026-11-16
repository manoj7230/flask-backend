# Flask Backend – Jenkins CI/CD

A small Flask API with a health endpoint and pytest test. Jenkins pipeline is defined in `Jenkinsfile` and deploys to an Ubuntu EC2 host using the systemd service `flask-backend`.

## Local run
```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt pytest
pytest -q
python app.py
```

Health endpoint: `GET /health` (port 5000 by default).

Configure the Jenkins job as Pipeline from SCM with Script Path `Jenkinsfile`. Review deployment paths and restricted sudo permissions before running against EC2.