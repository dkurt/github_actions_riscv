# GitHub Actions on RISC-V

This repository provides a guide on how to build and use GitHub Actions (GitHub CI) on Self-Hosted RISC-V CPU machines.

**Disclaimer**: this project is not related to GitHub or .NET development. Any instructions or binary packages published in the spirit of the open source. Use carefully.

## How to use

Use a [pre-compiled](https://github.com/dkurt/github_actions_riscv/releases) version or follow the build steps from [build.yaml](.github/workflows/build.yaml).
Cross-compilation is not supported for now so build process is performed on RISC-V board which takes about 1 hour.

After unpacking the runner archive, install .NET:
```bash
cd $HOME
wget https://github.com/dkurt/dotnet_riscv/releases/download/v8.0.101/dotnet-sdk-8.0.101-linux-riscv64.tar.gz

sudo mkdir /usr/share/dotnet
cd /usr/share/dotnet
sudo tar -xf $HOME/dotnet-sdk-8.0.101-linux-riscv64.tar.gz
```

Verify .NET installation:
```bash
./dotnet --info
```

Then do `./config.sh` and `./run.sh` as recommended in your repository `Settings->Actions->Runners->New self-hosted runner` tab.

Runner was tested on:

| Board | OS | Comment |
|-------|----|---------|
| MangoPi MQ-Pro | [Ubuntu 23.10](https://ubuntu.com/download/risc-v) | |
| Sipeed Lichee RV Dock | [Ubuntu 23.10](https://ubuntu.com/download/risc-v) | |
| Sipeed Lichee RV Dock | [20211230_LicheeRV_debian_d1_hdmi_8723ds.7z](./howto_setup_rvv.md) | Use [GCC version](https://github.com/dkurt/dotnet_riscv/releases) ot .NET because of `fence.tso` hardware bug  |

## Runner at system startup

1. Create a file `~/.config/systemd/user/actions.service` with content:

  ```
  [Unit]
  Description=Start GitHub Runner

  [Service]
  Type=simple
  ExecStart=/home/sipeed/actions/run.sh

  [Install]
  WantedBy=default.target
  ```
  where `/home/sipeed/actions` is a path to installed package.

2. Run commands:

  ```bash
  systemctl daemon-reload --user
  systemctl enable actions --user
  ```
