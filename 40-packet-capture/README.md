# Packet capture

```
curl -sL \
https://github.com/siemens/edgeshark/raw/main/deployments/wget/docker-compose.yaml | docker compose -f - up -d
```


## Edgeshark

### Windows

Windows users get to enjoy a simple installer-based workflow that installs the URL handler and the Wireshark plugin in one go.

Download the installer file - <https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_windows_amd64.zip>

Unzip the archive and launch the installer script.

### MacOs

MacOs users have to suffer a little. But it is not that bad either.

To install the URL handler paste the following in the Mac terminal app:

```bash
mkdir -p /tmp/pflix-handler && cd /tmp/pflix-handler && \
rm -rf packetflix-handler.zip packetflix-handler.app __MACOSX && \
curl -sLO https://github.com/srl-labs/containerlab/files/14278951/packetflix-handler.zip && \
unzip packetflix-handler.zip && \
sudo mv packetflix-handler.app /Applications
```

To install the extpcap wireshark plugin execute in the Mac terminal:

```bash
# for x86_64 MacOS use https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_darwin_amd64.tar.gz
DOWNLOAD_URL=https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_darwin_arm64.tar.gz
mkdir -p /tmp/pflix-handler && curl -sL $DOWNLOAD_URL | tar -xz -C /tmp/pflix-handler && \
open /tmp/pflix-handler && open /Applications/Wireshark.app/Contents/MacOS/extcap
```

The command above will open two Finder windows, one with the `cshargextcap` binary and the other with the Wireshark's existing plugins. Move the `cshargextcap` file over to the window with Wireshark plugins.

### Web UI

<http://ac1.srexperts.net:50500+ID>
ID   = 23
Port = 50523

Note, https schema is not enabled by default.
