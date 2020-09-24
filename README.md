
# Jest Coverage Guard

> Enforce code coverage on legacy code.


> Coverage quality guard will fail your tests if coverage in changed and added files is below a defined threshold
> Checks all locally committed and uncommitted files on current branch

## Usage

1. Install jest-coverage-guard in your project

	Install with npm:

	```bash
	npm install --save-dev jest-coverage-guard
	```

	Install with yarn:

	```bash
	yarn add jest-coverage-guard --dev
	```

2. Create config file (`coverage.guard.config.json`) in your project root:

      _(optional: "excludeKeywords",  "excludeFiles", "addedFilesQualityGate", "changedFilesQualityGate")_

    ```json
    {
      "appRootRelativeToGitRepo": "src",
      "excludeKeywords": [
        "// skip-coverage-check",
        "backbone",
        "class SomeUglyClass"
      ],
      "featureNameRegExp": {
        "body": "APP-[0-9]+",
        "flags": "g"
      },
      "excludeFiles": [ "-test.js", ".less"],
      "addedFilesQualityGate": {
        "statements": 100,
        "branches": 100,
        "functions": 100,
        "lines": 100
      },
      "changedFilesQualityGate": {
        "statements": 80,
        "branches": 80,
        "functions": 80,
        "lines": 80
      }
    }
    ```

2. In your jest config file `jest.config.json` add jest-coverage-guard as a reporter:
    `"reporters": ["default", "jest-coverage-guard"],`

4. Run jest tests


#### Config options

* **`appRootRelativeToGitRepo`**  - path to your application source code !important: this path should be relative to git repository root and not to CWD
* **`excludeKeywords`** - keywords that will be used to skip a file from coverage. If a file contains one of the keywords the coverage check script will ignore this file.
* **`featureNameRegExp`** - patters to search commits by commit message. Assuming you are adding in each commit message a issue/ticket number the the beginning. e.g: 'APP-12345 fixed a nasty bug'
* **`excludeFiles`** - array of file extensions that should be ignored from check
* **`addedFilesQualityGate`** - config object where you specify quality gate for newly added files, defaults to:
```json
{
  "statements": 100,
  "branches": 100,
  "functions": 100,
  "lines": 100
}
```

* **`changedFilesQualityGate`** - config object where you specify quality gate for changed  files, defaults to:
```json
{
  "statements": 80,
  "branches": 80,
  "functions": 80,
  "lines": 80
}
```

## Why use it?

	You can use this to fail your CI pipeline when code coverage is not good enough.
	This script should be executed only after jest generated the coverage report, so as jest custom reporter.
	This script with check your committed and uncommitted files.

  Locally the script will check coverage on all files that you have changed and committed
  implementing current feature and all files that you are currently working with
  but not yet committed.

  In CI environment it will only check committed files for this feature

  If you want to exclude your file from coverage_check for some reason (it should be in excludeKeywords config):
  add a comment to your file with "skip-coverage-check", eg. `// skip-coverage-check`

### How does it work:

  1. The script assumes that you are working with a project management software where you have ticket/issue
	numbers that are appended or included in every commit message, eg: "APP-12345: fixed typo in awesomefile.js"
  2. Script gets the name of the current branch (config: featureNameRegExp)
  3. Extracts the ticket number from the branch name eg. APP-12345 (cconfig: featureNameRegExp)
  4. Gets all commits that contain this ticket number in the message.
  5. Gets all url's of the files you have changed in those commits.
  6. Filters the file URL's to get only app files (config: appRootRelativeToGitRepo) and skips the files that contain exclude keywords (config: excludeKeywords).
  7. Finds report for each file from jest coverage report object.
  8. Compares the results with corresponding quality gate mask.
  9. Adds results to results table.
  10. Adds failed coverage errors to the errors table.
  11. Shows the result table.
  12. Shows errors table.
  13. If error table contains errors - fail the script with exit `code 1`.
  14. If error table contains error but you are locally in `--watch` or `--watchAll` mode - script will not fail.
  15. If error table contains no errors - script will succeed.

  		NEXT STEPS ONLY LOCALLY:
  16. Gets current git.status Object.
  17. Gets all files that you have changed but not yet committed.
  18. Gets file URL's and then does everything from step 6 to 15.
