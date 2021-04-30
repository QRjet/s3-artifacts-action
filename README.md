# Upload Artifacts to S3

The purpose of this action is to upload the zipped artifact from github actions and upload it to S3, segregated by repository and branch. 

This allows the organisation to package and upload all their artifacts to the same S3 bucket for deployment.

To be able to access these artifacts, the repository has to be sent to platform devops team. They add it to a Google OAuth app, to make the suddomain url accessbile to Medly users.

## Usage

This action requires 3 inputs: AWS Access Key Id, Secret Access Key, and the name of the zipped distribution file created by the github action.
You can give an optional  S3 Bucket Name, and  AWS Region. The defaults are that of our dev environment.
The following a snippet of how this can be used. You can also find the same example snippet is also available at ```.github/workflows/build.yml``` of this repository

```yaml

name: build

on: [push, pull_request]

jobs:
  publish-s3-artifact:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Upload to S3 as artifact
        uses: medlypharmacy/s3-artifacts-action@master
        with:
          aws_access_key_id: ${{ secrets.S3_BUILD_ARTIFACTS_ACCESS_KEY_ID}}
          aws_secret_access_key: ${{ secrets.S3_BUILD_ARTIFACTS_SECRET_ACCESS_KEY}}
          source_path: './dummy91.zip'
```

### Optional Parameters

You can use additional optional parameters depending on your usecase. You can specify a custom S3 bucket. You can also specify whether the upload needs to go to a custom path. Also, if the source and destination is a directory, you can specify the resource type as a directory. If nothing is specified, it is assumed that the DIST file is a zip file.

```yaml

publish-s3-artifact-folder:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Upload to S3 artifact folder
        uses: medlypharmacy/s3-artifacts-action@master
        with:
          aws_access_key_id: ${{ secrets.S3_BUILD_ARTIFACTS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.S3_BUILD_ARTIFACTS_SECRET_ACCESS_KEY }}
          aws_s3_bucket_name: ${{ secrets.S3_BUCKET_NAME }}
          source_path: './dummyfolder'
          destination_path: "/dummyartifacts"
          exclude_repo_from_destination_path: true
          resource_type: "DIRECTORY"  
```

### To generate SWAGGER to html

You can generate html docs for corresponding swagger specification files (.yml files). The source_path should be directory containing the swagger specification files. Providing destination_path for resource_type: "SWAGGER_TO_HTML" is not supported at this moment. The swagger docs would be available at:
https://<repo_name>.medly.dev/swagger-docs/<swagger_spec_file_name_without_extension>/index.html

For example, for this repo swagger docs are available at:

https://s3-artifacts-action.medly.dev/swagger-docs/dummy_swagger/index.html

https://s3-artifacts-action.medly.dev/swagger-docs/dummy_swagger1/index.html

```yaml
publish-s3-artifact-folder:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Upload to S3 artifact folder
        uses: medlypharmacy/s3-artifacts-action@master
        with:
          aws_access_key_id: ${{ secrets.S3_BUILD_ARTIFACTS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.S3_BUILD_ARTIFACTS_SECRET_ACCESS_KEY }}
          aws_s3_bucket_name: ${{ secrets.S3_BUCKET_NAME }}
          source_path: './api/spec'
          resource_type: "SWAGGER_TO_HTML"
```


### To publish test-coverage reports 

You can publish test-coverage reports generated by your test. source_path value would be the directory where the test-reports are generated. The test-coverage reports would be available at: https://<repo_name>.medly.dev/test-coverage/


```yaml
publish-s3-artifact-folder:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Upload to S3 artifact folder
        uses: medlypharmacy/s3-artifacts-action@master
        with:
          aws_access_key_id: ${{ secrets.S3_BUILD_ARTIFACTS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.S3_BUILD_ARTIFACTS_SECRET_ACCESS_KEY }}
          aws_s3_bucket_name: ${{ secrets.S3_BUCKET_NAME }}
          source_path: './build/reports/jacoco/codeCoverageReport/html'
          resource_type: "TEST_COVERAGE"
```
## Development

- Add dependency that is required by the action in Dockerfile
- Use the dependency in entrypoint.sh
- Create a branch, and raise a PR for any change
- Test the changes by following the below steps
  - Add test for any new feature added in the [workflow](.github/workflows/build.yml)
  - [action.yml](action.yml): Change the version of image used to point to your branch
    > `image: docker://ghcr.io/medlypharmacy/s3-artifacts-action/runner:<your-branch>`
  - [.github/workflows/build.yml](.github/workflows/build.yml)
    > Use this branch in workflow job 'test'
      ```
      name: Upload to S3 as artifact
      uses: medlypharmacy/s3-artifacts-action@<your-branch>
      ```
  - Revert these changes, and point to `master` again once the build passes
