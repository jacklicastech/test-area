## Building

1. Clone this project

        git clone git@github.com:appilee/luna-build

2. Init submodules

        git submodule init
        git submodule update

3. Build, passing manufacturer and device

        ./build.sh castles mp200
        ./build.sh castles mars1000


## Details

Most of `build.sh` just attempts to automate a few commands. First it makes
sure supporting tar files are in place; these are copied from S3 so you will
need `aws` CLI tools installed and you will need credentials. After the files
are verified it will update luna from git and create a tar file from it too,
copying it into the docker image directory.

Once all the preconditions are met, the build really just boils down to:

    # build the base image for the manufacturer, which is shared by all
    # device types
    docker build castles/base

    # build the docker image for the chosen device; this also builds luna
    # within the image
    docker build castles/mp200

    # copy the build artifacts from within the docker image to the local
    # file system, into tmp/
    id=$(docker create $name)
    docker cp $id:/tmp/src/luna/dist/output/application.cap ./tmp
    docker rm -v $id

    # send the build artifacts to S3. This step is skipped if you set
    # the NOS3 environment variable.
    aws s3 sync tmp s3://appilee/luna/tip/mp200/
