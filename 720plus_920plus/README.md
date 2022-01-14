# TensorFlow wheels for Synology 720+ and 920+

All the wheels listed here were built on DS720+ (Intel Celeron J4125) with gcc's flag -march=native. 
Synology DS920+ is equipped with the same processor.

## Build instructions

### Ubuntu 18.04 / tensorflow-1.15.5-cp36-cp36m-linux_x86_64.whl

```bash
docker run -itd --name tensorflow-builder -v "$PWD:/mnt" ubuntu:18.04 bash
docker attach tensorflow-builder

apt update && apt install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    curl \
    git \
    python3-dev \
    python3-setuptools \
    python3.6 \
    unzip

ln -s /usr/bin/python3.6 /usr/bin/python
curl https://bootstrap.pypa.io/get-pip.py | python3.6
pip3 install keras==2.3.0 "numpy<1.19.0" pandas
curl -O https://github.com/bazelbuild/bazel/releases/download/0.26.1/bazel-0.26.1-installer-linux-x86_64.sh
chmod +x bazel-0.26.1-installer-linux-x86_64.sh
./bazel-0.26.1-installer-linux-x86_64.sh --prefix=/opt/bazel
ln -s /opt/bazel/bin/bazel /usr/local/bin
git clone https://github.com/tensorflow/tensorflow.git --branch v1.15.5 --depth=1
cd /tensorflow
./configure
bazel build --verbose_failures --config=opt --config=v1 //tensorflow/tools/pip_package:build_pip_package
# [...]
# INFO: Elapsed time: 29780.459s, Critical Path: 405.13s (~ 8,27 h)
# INFO: 19489 processes: 19489 local.
# INFO: Build completed successfully, 20498 total actions

./bazel-bin/tensorflow/tools/pip_package/build_pip_package /mnt/
exit

docker rm -f tensorflow-builder
```

Tip: You can press `Ctrl+P Ctrl+Q` (`^P^Q` on Mac) to detach the container without interrupting the build proces. Type `docker attach tensorflow-builder` to go back.

Sources:
- https://www.tensorflow.org/install/source?hl=en
- https://github.com/evdcush/TensorFlow-wheels
- https://github.com/tensorflow/tensorflow/issues/41061#issuecomment-662222308
- https://github.com/tensorflow/tensorflow/issues/41584#issuecomment-662261561
- https://community.home-assistant.io/t/tensorflow-and-official-docker-image-cpu-instruction-issue/78310/26
