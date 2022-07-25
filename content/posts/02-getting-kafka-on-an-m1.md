---
title: Running a Python Client for Kafka on Apple Silicon/M1 
date: "2022-04-14"
description: How to install confluent-kafka on an M1.
draft: true
---


python packages `confluent-kafka`

`confluent-kafka` requires `librdkafka` and `brew install librdkafka` doesn’t seem to work, so you need to clone the repo and build it yourself.

### `Librdkafka`

```bash
git clone https://github.com/edenhill/librdkafka.git
cd librdkafka
./configure --install-deps
brew install openssl zstd pkg-config
brew link openssl --force
export PATH="/opt/homebrew/opt/openssl@3/bin:$PATH" # add to your .rc file
./configure
make
sudo make install
```

<aside>
💡 **Note:** I used the `/opt/homebrew/opt` path but maybe `/opt/homebrew/Cellar` makes more sense? Someonne should try it that way and update this tutorial if it works

</aside>

### `confluent-kafka`

```python
python -m pip install --global-option=build_ext --global-option="-I/usr/local/include" \
                      --global-option="-L/usr/local/lib" confluent-kafka

```           