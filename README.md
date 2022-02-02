# Dockerised TensorFlow on an Apple Silicon host (M1 chip)

## Introduction

There seems to be a known issue when running TensorFlow inside a Docker image that targets the `x86_64` architecture (the
one used in Intel 64bit chips):

- Main issue, worth following: https://github.com/tensorflow/tensorflow/issues/52845
- Related, affecting TensorFlow-serving: https://github.com/tensorflow/serving/issues/1948

The root cause seems to be that QEMU, which emulates the `x86_64` architecture on behalf of Docker, does not support AVX
instructions, which are used in the official releases of TensorFlow in PyPI. More details in the first issue.

One could follow one of the following fixes (the first two
mentioned [in this comment](https://github.com/tensorflow/tensorflow/issues/52845#issuecomment-1025015276), the last two
mine):

1. Run docker with `x86_64` using an unofficial wheel compiled without AVX, which will be slow.
2. Run docker with `arm64` (the architecture of M1 chips) using an unofficial wheel for this architecture, which will
   work fine as long as all other packages and utilities used in your project work with this architecture.
3. Throw the MacBook out of the window, get an old Intel-based MacBook or get a Linux laptop.
4. Wait until someone fixes this mess. Work on something else in the meantime.

Next, we present some simple examples of approaches 1 (tbd) and 2.

## Solution 1

TBD

## Example with solution 2

See the `requirements.txt` file, where one of three builds of TensorFlow is installed:

1. the official `tensorflow` for Linux environments

```
docker build --platform linux/x86_64 -f Dockerfile -t m1-support-x86_64 .
docker run --platform linux/x86_64 --rm m1-support-x86_64

>> Output when running on a `arm64` host:
>> x86_64
>> The TensorFlow library was compiled to use AVX instructions, but these aren't available on your machine.
>> qemu: uncaught target signal 6 (Aborted) - core dumped
>> Aborted
```

2. KumaTea's unofficial `arm64` `tensorflow` for Linux environments

```
docker build --platform linux/arm64 -f Dockerfile -t m1-support-arm64 .
docker run --platform linux/arm64 --rm m1-support-arm64

>> Output in `arm64` host:
>> aarch64
>> tf.Tensor(934.2583, shape=(), dtype=float32)
```

3. `tensorflow-macos` for macOS systems (don't forget to use a virtualenv!):

```
pip install -r requirements.txt
python test.py

>> Output in `x86_64` host:
>> arm64
>> tf.Tensor(-89.86255, shape=(), dtype=float32)
```