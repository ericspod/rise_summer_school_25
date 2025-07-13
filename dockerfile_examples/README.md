# Dockerfile Examples

These are a few Dockerfile examples you can look at and build to see how a few techniques are done in practice.

## Cowsay

This is another variation of the Cowsay example. Here the repository is checked out and used directly rather than
install with pip. This also uses Alpine as a base image, which is a very small distribution suited for very simple
tasks that don't need a full Linux system like Ubuntu. It does use base libraries that aren't compatible with CUDA
so you can't really use GPUs in them.

Build with the usual command but specify the Dockerfile explicitly:

`docker build . -t cowsaypy -f Dockerfile.cowsay`

The `ENTRYPOINT` command in the file specifies running the cowsay module as a program but doesn't provide all the 
arguments needed to print something to screen. The caller needs to provide further arguments to define a message and 
optionally change the character, eg.:

`docker run --rm cowsaypy -t "Hello, Alpine!' -c tux`

The usage can be printed with `docker run --rm cowsaypy --help`.

## Simple Input/Output

This is also built off Alpine but installs the tree program. It will examine the contents of an input directory and
put the tree output into a text file in the output directory. Building:

`docker build . -t in_out -f Dockerfile.in_out`

The idea is to mount a directory in the host system and get the tree output of its contents. One quirk of Docker is that
it runs as root internally, and so output files are owned by root which is problematic at times. One solution is to run
as the current user/group with the `-u` flag:

```sh
mkdir out
docker run --rm -u $(id -u):$(id -g) -v $(pwd):/input -v $(pwd)/out:/output in_out
```

This will write to `./out/tree.txt` which is owned by the current user (as given by the user ID `id -u`). 

The input/output paths are defined by environment variables which can be modified on the command line. Instead of 
mounting `/input`, the variable for it can be changed with `-e` to point somewhere in the running container, eg. `/etc`:

`docker run --rm -u $(id -u):$(id -g) -e INPUT_DIR=/etc -v $(pwd)/out:/output in_out`

The `./out/tree.txt` file will now contain a tree listing of `/etc`.

## Minimum PyTorch

This builds on the slim Python 3.12 image by installing PyTorch through pip. All the libraries needed for this are 
pulled down which adds up to almost 6GiB, but this is still small for a Docker image. Building is as such:

`docker build . -t min_pt -f Dockerfile.min_pt`

Running the container without modification just uses the placeholder `app.py` file, so this should be changed/replaced
to include project code. To test that the PyTorch installation can create a tensor on a GPU, use the following:

`docker run --rm --gpus all min_pt python -c 'import torch;print(torch.rand(4).to("cuda:0"))'`

This will print a tensor indicating which device it is on. This works by running Python directly with arguments passed
to it, to get a bash terminal in the running container use:

`docker run --rm -ti --gpus all min_pt /bin/bash`

## MONAI Training and Inference Example

This example repository provides the training and inference scripts for a MONAI classifier: https://github.com/ericspod/MONAI_101

The Dockerfile in this repo demonstrates installing requirements with pip in an efficient way and how to run inference
with a mounted directory of inputs. It also uses the slim Python base image but installs PyTorch as a dependency of 
MONAI, so the version it gets will be different from what's in the previous example image.

One thing to note is that it copies the contents of the repo into the image with `COPY . /app`. Instead, one can clone 
the repo into the `/app`  directory with `git clone https://github.com/ericspod/MONAI_101 /app` and then the following 
commands can run as normal. This allows a Dockerfile to be defined without local data, however it does man that `git`
needs to be installed in the image and the .git directory will be present if not deleted explicitly. 
