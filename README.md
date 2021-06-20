# Source-compiled-TensorFlow-on-aarch64 with GCC-11

# Building TensorFlow from source (TF 2.5.0, Cray-centos8-aarch64 (Apollo80), GCC-11.1.0)

## Why build from source?
The official instructions on installing TensorFlow are here: https://www.tensorflow.org/install.
If you want to install TensorFlow just using [pip](https://www.tensorflow.org/install/pip), you are running a supported Ubuntu LTS distribution, and you're happy to install the respective tested CUDA versions (which often are outdated), by all means go ahead. A good alternative may be to run a [Docker image](https://www.tensorflow.org/install/docker).

I am usually unhappy with installing what in effect are pre-built binaries. The prenuilt binaies for our architecture was not available while writing this post.

So installing TensorFlow from source becomes a necessity. The official instructions on building TensorFlow from source are here: https://www.tensorflow.org/install/install_sources.

What they don't mention there is that on supposedly "unsupported" configurations (i.e. up-to-date Linux systems), this can be a task from hell. So far, building TensorFlow has been a mostly terrible experience. With some TensorFlow versions and combination of system properties (OS version, CUDA version), things may work out decently. But more often than not, issues have popped up during the many times I have tried to build TensorFlow.

## Described configuration

I am describing the steps necessary to build TensorFlow in (currently) the following configuration:

 - centos8 
  - GCC 11.1.0 
  - TensorFlow v2.5.0

At the time of writing (**2021-20-06**), these were the latest available versions. 


## Prerequisites

I have installed Prerequisites with SPACK utility available at - https://github.com/spack/spack

The respective modules for me were-


module load  bazel-3.7.2-gcc-11.1.0-opk2wx2                    
module load  py-mpi4py-3.0.3-gcc-11.1.0-ui3ugpd
module load  gcc-11.1.0-gcc-10.2.0-2be3kjt                     
module load  py-protobuf-3.12.2-gcc-11.1.0-tqnhu3u
module load  openblas-0.3.15-gcc-11.1.0-py7rpuc  
module load  py-scipy-1.6.3-gcc-11.1.0-vqtczgq
module load  python-3.9.5-gcc-11.1.0-uxuyerb                   
module load  py-setuptools-51.0.0-gcc-11.1.0-sgp2x7j
module load  openjdk-11.0.6_9-201912311630-gcc-11.1.0-fnj22np 
module load  py-six-1.15.0-gcc-11.1.0-zbw4glm
module load  mpich-3.4.2-gcc-11.1.0-yv33elt
module load  flatbuffers-1.12.1-gcc-11.1.0-aixfvgp
module load  py-bottleneck-1.3.2-gcc-11.1.0-xex4agg
module load  py-future-0.18.2-gcc-11.1.0-tyqhkr7
module load  py-h5py-2.10.0-gcc-11.1.0-xqy7x6d
module load  py-numpy-1.20.3-gcc-11.1.0-y4bqjos
module load py-grpcio-1.32.0-gcc-11.1.0-ljawnoo
module load  py-keras-applications-1.0.8-gcc-11.1.0-uzhmqle
module load  py-keras-preprocessing-1.1.2-gcc-11.1.0-ydrwhvl


## Building TensorFlow

### Cloning and patching

First clone the sources, and check out the desired branch. At the time of writing, `v2.2.0` was the latest version; adjust if necessary.

      $ git clone https://github.com/tensorflow/tensorflow
      $ cd tensorflow
      $ git checkout v2.5.0

### Configuration

Now run the TensorFlow configuration script

      $ ./configure

configure script will ask certain questions, read them carefully and decide your configuration.


### Building

Now we can start the TensorFlow build process for default configuration. 

	$ bazel build --config=opt -c opt //tensorflow/tools/pip_package:build_pip_package
	
I have made my own bazel build command, tried several combinations, putting here only combination which worked-
#Customised bazel command which worked for me on aarch64 (apollo80) machine -
#Tried building with oneDNN, waiting for fix for GCC-11. PR #1097 to be merged.

	$bazel build  --local_cpu_resources="HOST_CPUS*.5" --local_ram_resources="HOST_RAM*.5" --config=v2 --config=nogcp --config=nonccl //tensorflow/tools/pip_package:build_pip_package

    
* Add `-c dbg --strip=never` in case you do not want debug symbols to be stripped (e.g. for debugging purposes).
    Usually, you won't need to add this option.
    
* Add `--compilation_mode=dbg` to build in debug instead of release mode, i.e. without optimizations.
    You shouldn't do this unless you really want to.

This will take some time. Have a coffee, or two, or three. Cook some dinner. Watch a movie.

### Building & installing the Python package

Once the above build step has completed without error, the remainder is now easy. Build the Python package, which the `build_pip_package` script puts into a specified location.

      $ ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

And install the build wheel package:

      $ pip install /tmp/tensorflow_pkg/tensorflow-2.3.0-cp38-cp38-linux_x86_64.whl

### Testing the installation

Google suggests to test the TensorFlow installation with the following command:

    $ python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

This does not make explicit use of CUDA yet, but will emit a whole bunch of initialization messages that can give an indication whether all libraries could be loaded. And it should print that requested sum.

It worked? Great! Enjoy!!!
