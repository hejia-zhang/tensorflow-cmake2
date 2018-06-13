# tensorflow-cmake2
Build TensorFlow Cpp application using CMake. This project is inspired by [tensorflow-cmake](https://github.com/cjweeks/tensorflow-cmake).

# Support Platform
|        OS        | TensorFlow | NVIDIA Driver | Bazel |  gcc | Protobuf | cuda  | cuDNN|
|:----------------:|:----------:| :-----------: | :----:|:----:|:--------:| :---: |:----:|
|Ubuntu16.04-x86_64|     1.8    |     387.26    | 0.13.0| 5.4.0| 3.6.0    | 9.1   |7.1.3 |

# Compile TensorFlow Cpp shared library
## Make a virtual environment for TensorFlow using Anaconda
```
$ conda create -n TFEnv python=3.6.4 # create a conda environment named TFEnv
```
**We will do following work in this environment**
```
$ source activate TFEnv # activate the environment
```
## Install dependencies
- Follow the [instructions](https://docs.bazel.build/versions/master/install-ubuntu.html) for installing Bazel.
- Using apt for installing following packages
    ```
    $ sudo apt install autoconf automake libtool curl make g++ unzip swig
    ```
- Using pip for installing following packages in conda virtual environment
    **Please make sure we are already in conda virtual environment**
    ```
    $ pip install numpy wheel dev
    ```
## Clone TensorFlow from its github repo
```
$ git clone https://github.com/tensorflow/tensorflow 
```
- Enter the cloned repo
- Checkout branch r1.8
    ```
    $ git checkout r1.8
    ```
## Compile Tensorflow Cpp shared library
- Based on the current tensorflow/BUILD file, we will get two shared libraries named libtensorflow_cc.so, 
libtensorflow_framework.so which include all the required dependencies for integration with a Cpp project. 
Following are the produce rule we will see in tensorflow/BUILD file
    ```
    tf_cc_shared_object(
        name = "libtensorflow_framework.so",
        framework_so = [],
        linkstatic = 1,
        visibility = ["//visibility:public"],
        deps = [
            "//tensorflow/core:framework_internal_impl",
            "//tensorflow/core:lib_internal_impl",
            "//tensorflow/core:core_cpu_impl",
            "//tensorflow/stream_executor:stream_executor_impl",
            "//tensorflow/core:gpu_runtime_impl",
        ] + tf_additional_binary_deps(),
    )
    
    tf_cc_shared_object(
        name = "libtensorflow_cc.so",
        linkopts = select({
            "//tensorflow:darwin": [
                "-Wl,-exported_symbols_list",  # This line must be directly followed by the exported_symbols.lds file
                "$(location //tensorflow:tf_exported_symbols.lds)",
            ],
            "//tensorflow:windows": [],
            "//tensorflow:windows_msvc": [],
            "//conditions:default": [
                "-z defs",
                "-s",
                "-Wl,--version-script",  #  This line must be directly followed by the version_script.lds file
                "$(location //tensorflow:tf_version_script.lds)",
            ],
        }),
        deps = [
            "//tensorflow:tf_exported_symbols.lds",
            "//tensorflow:tf_version_script.lds",
            "//tensorflow/c:c_api",
            "//tensorflow/c/eager:c_api",
            "//tensorflow/cc:cc_ops",
            "//tensorflow/cc:client_session",
            "//tensorflow/cc:scope",
            "//tensorflow/cc/profiler",
            "//tensorflow/core:tensorflow",
        ],
    )
    ```
- Build the shared library and copy them to /usr/local/lib
    ```
    # Note that this requires user input
    # Use virtual env python interp
    $ ./configure 
    ```
    - For gpu users:
        ```
        $ bazel build --config=opt --config=cuda --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" tensorflow:libtensorflow_cc.so tensorflow:libtensorflow_framework.so
        ```
    ```
    $ sudo cp bazel-bin/tensorflow/{libtensorflow_cc.so,libtensorflow_framework.so} /usr/local/lib 
    ``` 
- Copy the source to /usr/local/include/google and remove unneeded items:
    ```
    $ sudo mkdir -p /usr/local/include/google/tensorflow
    $ sudo cp -r tensorflow /usr/local/include/google/tensorflow
    $ sudo find /usr/local/include/google/tensorflow/tensorflow -type f  ! -name "*.h" -delete
    ```
- Copy all generated files from bazel-genfiles:
    ```
    $ sudo cp bazel-genfiles/tensorflow/core/framework/*.h  /usr/local/include/google/tensorflow/tensorflow/core/framework
    $ sudo cp bazel-genfiles/tensorflow/core/lib/core/*.h  /usr/local/include/google/tensorflow/tensorflow/core/lib/core
    $ sudo cp bazel-genfiles/tensorflow/core/protobuf/*.h  /usr/local/include/google/tensorflow/tensorflow/core/protobuf
    $ sudo cp bazel-genfiles/tensorflow/core/util/*.h  /usr/local/include/google/tensorflow/tensorflow/core/util
    $ sudo cp bazel-genfiles/tensorflow/cc/ops/*.h  /usr/local/include/google/tensorflow/tensorflow/cc/ops
    ```
- Copy the third party directory
    ```
    sudo cp -r third_party /usr/local/include/google/tensorflow/
    sudo rm -r /usr/local/include/google/tensorflow/third_party/py
    ```
# Test





