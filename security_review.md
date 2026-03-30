**Quick Start:** Next time AI recommends a package, open this file and run through the 3 steps before you install. The terminal commands in Step 4 do the heavy lifting — copy-paste them directly. 
Or drop this into claude code and ask it go create a /security-review skill.


---


Run this BEFORE you install, deploy, or integrate any tool recommended by AI, a curated list, a blog, or a social media post.


## Step 1: Source Check


| Question | Green Flag | Red Flag |
|----------|-----------|----------|
| Who wrote the recommendation? | Named person with a verifiable profile | Anonymous blog, SEO content farm, no byline |
| Where did you find it? | Official docs, reputable tech publication, peer recommendation | "Awesome" list, marketing blog, sponsored post |
| Can you find 3+ independent sources confirming it works? | Yes — different authors, different sites | Only the tool's own marketing + one curated list |


Rule: A curated repo with 30K stars does not validate individual tools listed in it. Each tool needs its own verification.


## Step 2: Package Check


| Question | How to Check | Red Flag |
|----------|-------------|----------|
| Does it exist on the official registry? | `npm view <package>` or `pip show <package>` | Package name doesn't exist or is subtly misspelled (typosquatting) |
| Who maintains it? | Check the GitHub repo -> contributors tab | Single anonymous maintainer, no commit history, forked from archived repo |
| When was the last commit? | GitHub repo -> commit history | No commits in 6+ months, open issues piling up with no responses |
| How many weekly downloads? | npmjs.com or pypistats.org | < 100/week for a supposedly popular tool |
| Any known vulnerabilities? | `npm audit` or `pip-audit` or Snyk | Known CVEs with no fix available |


Rule: Popular packages get compromised too (event-stream had 2M weekly downloads when it was hijacked). Downloads alone don't mean safe.


## Step 3: Method Check


| Question | Green Flag | Red Flag |
|----------|-----------|----------|
| Does it use official/documented APIs? | Uses published REST API, SDK, or OAuth flow | Uses private API, undocumented endpoints, or browser automation to bypass login |
| Does it comply with the service's ToS? | Explicitly mentions API terms compliance | "Works great!" with no mention of how it accesses the service |
| Does it require your credentials? | OAuth token with scoped permissions | Asks for your username + password directly |
| What permissions does it request? | Only what it needs for its stated function | Broad permissions (full account access for a tool that only needs read) |


Rule: If a tool gives you access to data the platform's official API doesn't expose, it's probably violating ToS. "Works today, banned tomorrow."


## Step 4: Quick Terminal Checks (Copy-Paste Ready)


```bash
# Check npm package exists and see metadata
npm view <package-name> name version author license homepage


# Check pip package
pip show <package-name>


# Check GitHub repo health
gh repo view <owner/repo> --json stargazersCount,updatedAt,licenseInfo,isArchived


# Check for known vulnerabilities
npm audit --package-lock-only  # for JS
pip-audit                       # for Python


# Check download trends
# Visit: npmjs.com/package/<name> or pypistats.org/packages/<name>
```


## The Scorecard


Score 1 point for each green flag across all 3 steps.


- **8-10 green flags:** Probably safe. Proceed.
- **5-7 green flags:** Investigate further before installing.
- **Below 5:** Skip it. Find an alternative.


AI recommends. You verify. Every single time.
