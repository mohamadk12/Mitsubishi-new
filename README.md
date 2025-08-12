
#!/usr/bin/env python3
"""
daily_commit.py
Create or update `daily-activity.md` in a GitHub repository to record a daily activity entry.
Requires environment variable GITHUB_TOKEN with a personal access token that has repo scope.
"""

import os
import base64
import json
import datetime
import requests
import sys
import random

# ------------- CONFIG -------------
GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN")
OWNER = "your-github-username"      # e.g. "alice"
REPO = "your-repo-name"             # e.g. "my-repo"
FILE_PATH = "daily-activity.md"
BRANCH = "main"                     # branch to commit to
# -----------------------------------

if not GITHUB_TOKEN:
    print("Error: Please set the GITHUB_TOKEN environment variable.")
    sys.exit(1)

API_BASE = f"https://api.github.com/repos/{OWNER}/{REPO}/contents/{FILE_PATH}"
HEADERS = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github.v3+json",
}

def fetch_file():
    """Return (sha, decoded_content) if file exists, or (None, "") if not."""
    resp = requests.get(API_BASE + f"?ref={BRANCH}", headers=HEADERS)
    if resp.status_code == 200:
        data = resp.json()
        sha = data.get("sha")
        content_b64 = data.get("content", "")
        content = base64.b64decode(content_b64.encode()).decode('utf-8')
        return sha, content
    elif resp.status_code == 404:
        return None, ""
    else:
        raise RuntimeError(f"Failed to fetch file: {resp.status_code} {resp.text}")

def make_entry():
    """Generate a short daily entry."""
    now = datetime.datetime.utcnow().replace(microsecond=0).isoformat() + "Z"
    messages = [
        "Daily update: did something productive ✅",
        "Committed a tiny improvement 🚀",
        "Keeping the streak alive — one day at a time 🔥",
        "Automated daily note — progress matters.",
        "Small step forward recorded."
    ]
    note = random.choice(messages)
    entry = f"- {now} — {note}\n"
    return entry

def put_file(new_content_b64, commit_message, sha=None):
    payload = {
        "message": commit_message,
        "content": new_content_b64,
        "branch": BRANCH,
    }
    if sha:
        payload["sha"] = sha
    resp = requests.put(API_BASE, headers=HEADERS, data=json.dumps(payload))
    if resp.status_code in (200, 201):
        return resp.json()
    else:
        raise RuntimeError(f"Failed to create/update file: {resp.status_code} {resp.text}")

def main():
    try:
        sha, existing = fetch_file()
    except Exception as e:
        print("Error while fetching file:", e)
        sys.exit(1)

    entry = make_entry()

    if existing:
        # Append entry to top (or bottom). We'll prepend to keep newest on top.
        new_text = f"# Daily Activity Log\n\n{entry}{existing}"
        commit_msg = f"Update daily-activity: {entry.strip()}"
    else:
        new_text = f"# Daily Activity Log\n\n{entry}"
        commit_msg = f"Create daily-activity: {entry.strip()}"

    new_b64 = base64.b64encode(new_text.encode('utf-8')).decode('utf-8')

    try:
        result = put_file(new_b64, commit_msg, sha=sha)
    except Exception as e:
        print("Error while writing file:", e)
        sys.exit(1)

    html_url = result.get("content", {}).get("html_url") or result.get("content", {}).get("download_url")
    print("Success! File committed. URL:", html_url)

if __name__ == "__main__":
    main()
