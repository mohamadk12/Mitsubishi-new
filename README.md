⁸Mon

#!/usr/bin/env python 
""
get_issues.py.type 
Fetch all open and closed issues from a GitHub repository and save them to issues_repord v
Requires environment variable GITHUB_TOKEN with repo read access.
"""


import r
import request

# ----------------- CONFIG ------------------


GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN") or "your_personal_access_token_here"
OWNER = "your-github-username"   # e.g. "alice"
REPO = "your-repo-name"          # e.g. "my-repo"
# -------------------------------------------

API_URL = f"https://api.github.com/repos/{OWNER}/{REPO}/issues"
HEADERS = {"Authorization": f"token {GITHUB_TOKEN}"}

def fetch_all_issues():
    issues = []
    page = 1
    while True:
        params = {"state": "all", "per_page": 100, "page": page}
        r = requests.get(API_URL, headers=HEADERS, params=params)
        if r.status_code != 200:
            raise RuntimeError(f"GitHub API error: {r.status_code} {r.text}")
        data = r.json()
        if not data:
            break
        issues.extend(data)
        page += 1
    return issues

def save_markdown(issues):
    with open("issues_report.md", "w", encoding="utf-8") as f:
        f.write(f"# GitHub Issues Report for {OWNER}/{REPO}\n\n")
        for issue in issues:
            number = issue["number"]
            title = issue["title"]
            state = issue["state"]
            url = issue["html_url"]
            f.write(f"- [#{number}]({url}) **{title}** — _{state}_\n")

def main():
    issues = fetch_all_issues()
    print(f"Fetched {len(issues)} issues.")
    save_markdown(issues)
    print("Saved to issues_report.md")

if __name__ == "__main__":
    main()
