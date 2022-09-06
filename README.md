name: Release
on: push
jobs:
  release-docker-helm-chart:
    name: Release Docker Image & Helm Chart
    timeout-minutes: 90
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Login to container registry
        run: |
          REPO_OWNER=`echo "${{ github.repository_owner }}" | tr '[:upper:]' '[:lower:]'`
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          
      - name: Build & Push Docker Image
        run: |
          REPO_OWNER=`echo "${{ github.repository_owner }}" | tr '[:upper:]' '[:lower:]'`
          docker build -t ghcr.io/${REPO_OWNER}/my-app:1.0.0 -f ./Dockerfile .
          docker push ghcr.io/${REPO_OWNER}/my-app:1.0.0
      - name: Pushing Helm Chart
        run: |
          REPO_OWNER=`echo "${{ github.repository_owner }}" | tr '[:upper:]' '[:lower:]'`
          # create chart package for chart `my-app`
          helm package my-app
          # Get packed chart file name
          PKG_NAME=`ls *.tgz`
          # pushing `my-app-1.0.0.tgz` to registry
          helm push ${PKG_NAME} oci://ghcr.io/${REPO_OWNER}/charts
