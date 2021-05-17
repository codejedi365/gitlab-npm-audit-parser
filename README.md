# GitLab parser for NPM Audit

    Usage: gitlab-npm-audit-parser [options]

    Input: Stdin via pipe
      npm audit --json | gitlab-npm-audit-parser ...
      cat <file> | gitlab-npm-audit-parser ...

    Options:

      -V, --version     output the version number
      -o, --out <path>  output filename, defaults to gl-dependency-scanning-report.json
      -h, --help        output usage information

## Package Objective

Perform the data translation from an `npm audit --json` report output to the
GitLab.com standardized JSON schema format specifically for ingest of dependency
scanning reports of a project.

## Why?

GitLab requires a common schema to ingest scanning reports from multiple
different dependency auditing tools across different languages. In the
JavaScript/TypeScript ecosystem, most of us use `npm audit` to verify project
dependencies but the JSON report is not ingestable by GitLab.com. It requires
this package as middleware to translate an `npm audit --json` report into the
standard dependency audit schema before it can be uploaded and ingested as a
dependency_scanning artifact. Ingested artifacts can then be used as data
sources to generate interactive content embedded in a pipeline results view or
Merge Request (MR) webpage.

## Compatibility

| INGEST                  | SUPPORTED? | OUTPUT                                      |
| ----------------------- | :--------: | ------------------------------------------- |
| npm-audit-report@^1.0.0 |    yes     | JSON file (gitlab-org/schema-merge@^14.0.1) |
| npm-audit-report@^2.0.0 |    yes     | JSON file (gitlab-org/schema-merge@^14.0.1) |

## How to use

Install this package into your devDependencies or use `npx` directly to download
the package at runtime. If you opt to download for use at run time, make sure to
include the correct scope name for the package since there are multiple versions
of this package on npmjs.com.

_I recommend the runtime option since this package is only needed in a GitLab
specific pipeline and not necessary to be locally installed for developer use._

```sh
# 1. Downloads at runtime use
npm audit --json | npx @codejedi365/gitlab-npm-audit-parser -o gl-dependency-scanning.json

# 2. Install in devDependencies
npm install --save-dev @codejedi365/gitlab-npm-audit-parser
```

Add the following job to `.gitlab-ci.yml`. If you used #2 and it is in your
devDependencies you may remove the @&lt;scope> prefix from the following.

```yaml
dependency scanning:
  image: node:10-alpine
  script:
    - npm ci
    - npm audit --json | npx @codejedi365/gitlab-npm-audit-parser -o
      gl-dependency-scanning.json
  artifacts:
    reports:
      dependency_scanning: gl-dependency-scanning.json
```

NOTE: If you use a `npm run-script` to call `npm audit` You must add the option
`--silent` to `npm run` or have `.npmrc` set the NPM loglevel to silent
otherwise the shell output will conflict with the stdin piping to this parser
and cause an error.

## Test

```sh
# Build CLI bundle & Execute all test cases
npm test
```

### Examples

| #   | INGEST FILE             |     | OUTPUT FILE                         |
| --- | ----------------------- | --- | ----------------------------------- |
| 1.  | `./test/v1_report.json` | =>  | `./test/snapshots/GL-report.1.json` |
| 2.  | `./test/v2_report.json` | =>  | `./test/snapshots/GL-report.2.json` |

## Future (Help Wanted)

- Configure a bot to monitor changes/updates to schema repository

- Implement a verification tool that interprets the schema and evaluates the
  generated report for accuracy

## Extras

**COMING SOON!**
[gitlab-depscan-merger](https://github.com/codejedi365/gitlab-depscan-merger): a
solution to create 1 ingestable dependency_scanning report from multiple audit
reports overcoming the GitLab pipeline limitation.

Check out my other projects at [@codejedi365](https://github.com/codejedi365) on
GitHub.com
