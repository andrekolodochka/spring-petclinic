This is a fork of Spring PetClinic Sample Application with a few changes.

1. The build was failing due to Postgres errors. To avoid that, [two tests](https://github.com/andrekolodochka/spring-petclinic/blob/main/src/test/java/org/springframework/samples/petclinic/PostgresIntegrationTests.java#L81) were removed.
2. Added a Dockerfile
3. Added a new frog.yml workflow:
- added a few environment variables on workflow level:
```env:
  JF_URL: ${{ vars.JF_URL }}
  JF_ACCESS_TOKEN: ${{ secrets.JF_ACCESS_TOKEN }}
  JFROG_CLI_AVOID_NEW_VERSION_WARNING:  ${{ vars.JFROG_CLI_AVOID_NEW_VERSION_WARNING }}
  IMAGE_NAME: petclinictest.jfrog.io/petclinic-docker/petclinic-docker-image:${{ github.run_number }}
  ```

- The instructions for `jfrog-setup-cli` action were not quite correct, so [created a PR](https://github.com/jfrog/setup-jfrog-cli/pull/114) to update those
- The workflow will:
  - build, tag and upush the image
  - scan image with XRay, the scan results are uploaded as artifacts to each workflow run
  - publish the build to Artifactory
4. Added a CodeQL workflow to scan the code. Scan results are uploaded as artifact to the workflow run