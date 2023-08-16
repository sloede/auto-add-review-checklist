# Auto-add review checklist to PRs
Test repo for automatically adding a review checklist to newly created PRs.

Currently, two variants are supported, both facilitated by the use of GitHub Actions: either
the review checklist is added as a *new comment* to each newly created PR, or it is
*appended to the description* of the PR.

## Usage

First, add your Markdown-formatted review checklist as the file `.github/review-checklist.md`
to your repository. An example checklist can be found [here](.github/review-checklist.md).

Next, create a file `.github/workflows/ReviewChecklist.yml` that will contain the workflow
for automatically adding the review checklist to new PRs. For the contents, you have two
options, as described in the following two subsections.

### Adding a review checklist as a new comment
Create the workflow file with the following content:
```yml
name: Add review checklist

on:
  pull_request_target:
    types: [opened]

jobs:
  add-review-checklist:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v3
      - name: Read review checklist template from file
        run: |
          {
            echo 'REVIEW_CHECKLIST<<REVIEW_CHECKLIST_EOF'
            cat .github/review-checklist.md
            echo
            echo 'REVIEW_CHECKLIST_EOF'
          } >> "$GITHUB_ENV"
      - name: Add review checklist as comment
        uses: actions/github-script@v6
        with:
          script: |
            const { REVIEW_CHECKLIST } = process.env;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: REVIEW_CHECKLIST
            });
```

### Appending the review checklist to the PR description
Create the workflow file with the following content:
```yml
name: Add review checklist

on:
  pull_request_target:
    types: [opened]

jobs:
  add-review-checklist:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v3
      - name: Read review checklist template from file
        run: |
          {
            echo 'REVIEW_CHECKLIST<<REVIEW_CHECKLIST_EOF'
            cat .github/review-checklist.md
            echo
            echo 'REVIEW_CHECKLIST_EOF'
          } >> "$GITHUB_ENV"
      - name: Append review checklist to PR description
        uses: actions/github-script@v6
        with:
          script: |
            const { REVIEW_CHECKLIST } = process.env;

            const result = await github.rest.pulls.get({
              pull_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
            });

            const sep = '\n\n---\n'
            const with_love = '\n\n*Note: review checklist added automatically with :heart: by Trixi.jl*';
            const body_updated = result.data.body + sep + REVIEW_CHECKLIST + with_love;

            github.rest.pulls.update({
              pull_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body_updated,
            });
```

## Author
This repository was initiated by [Michael Schlottke-Lakemper](https://lakemper.eu).

## License
This repository is licensed under the MIT license (see [LICENSE.md](LICENSE.md)).