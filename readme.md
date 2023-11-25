This is a fork of Spring PetClinic Sample Application with a few changes.

# Dockerfile
The Dockerfile is needed to build the image, but the file is missing. Added a [Dockerfile](https://github.com/andrekolodochka/spring-petclinic/blob/main/Dockerfile)

# Workflows

## Build and Test with Maven - maven-build.yml

The build in [maven-build.yml](https://github.com/andrekolodochka/spring-petclinic/blob/main/.github/workflows/maven-build.yml) was failing due to Postgres errors. To avoid that, [two tests](https://github.com/andrekolodochka/spring-petclinic/blob/main/src/test/java/org/springframework/samples/petclinic/PostgresIntegrationTests.java#L81) were removed.

## Building, scanning and publishing Docker image to Artifactory - jfrog.yml
Added a new [jfrog.yml](https://github.com/andrekolodochka/spring-petclinic/blob/main/.github/workflows/jfrog.yml) workflow
- added a few environment variables on workflow level:
```env:
  JF_URL: ${{ vars.JF_URL }}
  JF_ACCESS_TOKEN: ${{ secrets.JF_ACCESS_TOKEN }}
  JFROG_CLI_AVOID_NEW_VERSION_WARNING:  ${{ vars.JFROG_CLI_AVOID_NEW_VERSION_WARNING }}
  IMAGE_NAME: petclinictest.jfrog.io/petclinic-docker/petclinic-docker-image:${{ github.run_number }}
  ```
`JF_UR` and `JFROG_CLI_AVOID_NEW_VERSION_WARNING` are stored as variables in the repository:
![image](https://github.com/andrekolodochka/spring-petclinic/assets/59625655/62640cc8-9b0d-4b6a-8f75-d04a5e1558c9)
`JF_ACCESS_TOKEN` is stored as a secret in the repository:
![image](https://github.com/andrekolodochka/spring-petclinic/assets/59625655/551c0aae-f585-439c-bab2-78936ebad346)

- The instructions for [`jfrog-setup-cli` action](https://github.com/jfrog/setup-jfrog-cli) were not quite correct, so I [created a PR](https://github.com/jfrog/setup-jfrog-cli/pull/114) to update those
- The workflow will:
  - build, tag and push the image
  - scan image with XRay locally, the scan results are uploaded as artifacts to each [workflow run](https://github.com/andrekolodochka/spring-petclinic/actions/workflows/jfrog.yml)
  - publish the image to Artifactory

## Code scanning - codeql.yml
Added a [CodeQL workflow](https://github.com/andrekolodochka/spring-petclinic/blob/main/.github/workflows/codeql.yml) to scan the code. Scan results are uploaded as artifact to the [workflow run](https://github.com/andrekolodochka/spring-petclinic/actions/workflows/codeql.yml):
![image](https://github.com/andrekolodochka/spring-petclinic/assets/59625655/18abdfdd-75d5-4569-b890-f0a6f5df3cc4)

# How to run the application
Run the following (replace <build_number> with the build number you want to execute):
```
docker pull petclinictest.jfrog.io/petclinic-docker/petclinic-docker-image:<build_number>
docker run -p 127.0.0.1:8080:8080/tcp --pull=never  petclinictest.jfrog.io/petclinic-docker/petclinic-docker-image:<build_number>
```
The application will be available on http://localhost:8080

# Areas for improvement
- Postgres test fail during maven build. Need to investigate why so and fix. Most likely due to Postgres configuration.
- Running `docker pull` doesn't work with `latest` tag. Can probably fix with tagging/pushing with `petclinic-docker-image:latest` in addition to `petclinic-docker-image:${{ github.run_number }}`, but I suspect that's not how it should work as `hello-world` app picks up `latest` tag automatically
- For some reason `docker run` downloads all dependencies and compiles the application again. Not sure what the reason is, could be because GitHub runner is ubuntu on Intel and my laptop is on M2. But with Docker it should be irrelevant... ¯\\\_(ツ)\_/¯
- CodeQL results are visible from the Security tab only to repo owners. To share the results I uploaded the scan report as an artifact to the workflow run. In the past I used [GitHub Security Report action](https://github.com/marketplace/actions/github-security-report-action) to convert from SARIF to PDF, but it fails this time, so I uploaded SARIF only. Might need to ping Peter to look at and update the action.
