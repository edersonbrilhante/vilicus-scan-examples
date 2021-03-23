# Vilicus Scan Examples

## Examples

### GitHub Workflow
```yaml
name: Vilicus Scan Demo
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Maximize Build Space
        uses: easimon/maximize-build-space@master
        with:
          root-reserve-mb: 512
          swap-size-mb: 1024
          remove-dotnet: 'true'
          remove-android: 'true'
          remove-haskell: 'true'
            
      - name: Checkout branch
        uses: actions/checkout@v2

      - name: Build the Container image
        run: docker build -t localhost:5000/local-image:${GITHUB_SHA} . 
      
      - name: Vilicus Scan
        uses: edersonbrilhante/vilicus-github-action@main
        with:
          image: localhost:5000/local-image:${{ github.sha }}

      - name: Upload results to github security
        uses: github/codeql-action/upload-sarif@v1
        with:
          sarif_file: artifacts/results.sarif
```

### GitLab CI
```yaml
include:
  - remote: https://raw.githubusercontent.com/edersonbrilhante/vilicus-gitlab/main/Vilicus.gitlab-ci.yml  

remote_image:
  extends: .vilicus
  variables:
    IMAGE: python
  tags:
    - crosscicd

local_image:
  extends: .vilicus
  variables:
    IMAGE: localhost:5000/local-image:${CI_COMMIT_SHORT_SHA}
  script:
    - docker build -t localhost:5000/local-image:${CI_COMMIT_SHORT_SHA} .
    - ./run-job.sh
  tags:
    - crosscicd
```