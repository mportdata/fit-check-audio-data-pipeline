# Newscast Transcriber

An Apache Beam pipeline for downloading the latest episodes from a set of news podcast/RSS feeds and transcribing them with OpenAI Whisper.

The project is designed as a small, portable data/AI pipeline that can run locally or on Google Cloud Dataflow. It includes containerisation, Cloud Build configuration, Terraform-managed cloud resources, and pytest coverage for the core feed, episode, and transcription components.

## What It Does

1. Reads a JSON configuration of news feed names and RSS feed URLs.
2. Parses each feed and selects the latest episode.
3. Downloads episode audio into a local or GCS-backed `podcasts/` path.
4. Splits audio into chunks for transcription.
5. Uses a Hugging Face Whisper model to generate text.
6. Writes transcripts into a `transcriptions/` path.

The Beam pipeline parallelises work across feeds, so the same code can be run against a small local feed list or deployed as a Dataflow job.

## Repository Structure

```text
src/main.py              # Beam pipeline entry point
src/modules/feed.py      # RSS parsing and latest-episode lookup
src/modules/episode.py   # Download, chunking, transcription, and file handling
src/modules/models.py    # Whisper model wrapper
config/feeds.json        # Feed configuration
tests/                   # Pytest coverage for models, feeds, and episodes
Dockerfile               # Apache Beam Python SDK image with project dependencies
cloudbuild.yaml          # Cloud Build image build/push configuration
terraform/               # GCS buckets and Dataflow service account resources
scripts/                 # Local, Dataflow, and flex-template helper scripts
```

## Local Setup

Create a Python environment and install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Run the pipeline locally:

```bash
bash scripts/python_run_local.sh
```

By default, local outputs are written under:

```text
data/podcasts/
data/transcriptions/
```

## Configuration

Feeds are configured in `config/feeds.json` as a mapping of display name to RSS URL.

Example:

```json
{
  "BBC Minute": "http://wsrss.bbc.co.uk/bizdev/bbcminute/bbcminute.rss",
  "NPR News": "https://feeds.npr.org/500005/podcast.xml"
}
```

The pipeline accepts two custom options:

```text
--base-path       Local path or GCS bucket prefix for downloaded audio and transcripts
--feed-url-file   Local path or GCS path to the feed configuration JSON
```

## Tests

Run the test suite with:

```bash
pytest
```

The test suite covers:

- Whisper transcriber initialisation
- RSS feed object creation and latest-episode extraction
- Episode download status and idempotency
- Transcription file creation and retrieval

## Running On Google Cloud Dataflow

The repository includes the pieces needed to run the pipeline on Dataflow:

- `Dockerfile` builds from the Apache Beam Python SDK image.
- `cloudbuild.yaml` builds and pushes the container image.
- `terraform/` creates GCS buckets and a Dataflow service account.
- `scripts/gcloud_build_dataflow.sh` builds a Dataflow flex template.
- `scripts/gcloud_run_dataflow.sh` launches a Dataflow job from that template.

The provided scripts contain project-specific values. Before running them in a new GCP project, update:

- GCP project ID
- Region
- GCS bucket names
- Dataflow template path
- Feed configuration path

## Why This Project Exists

This project is a compact example of turning an AI model into an operational data pipeline:

- Batch processing with Apache Beam
- Cloud execution with Dataflow
- Containerised runtime dependencies
- Infrastructure-as-code for cloud resources
- Tests around the core pipeline objects
- A real-world input source with repeated, time-sensitive data

## Engineering Notes

- Beam keeps the feed processing model portable: the same transform structure can run locally during development or on Dataflow for cloud execution.
- The project separates feed parsing, episode handling, and transcription so each part can be tested independently.
- Containerisation and Terraform make the runtime and cloud resources explicit, which is the main engineering value of the project beyond the transcription model itself.
