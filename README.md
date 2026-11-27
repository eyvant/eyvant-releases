# Eyvant releases

Signed builds of the Eyvant apps and command-line tool, built by
`.github/workflows/build.yml` from a release tag of the source.

## Files

| File | What it is |
| --- | --- |
| `eyvant-app-android-arm64-v8a.apk` | The Android app |
| `eyvant-app-windows-x64-setup.exe` | The Windows app's installer. It is not code-signed, so SmartScreen warns on first run |
| `eyvant-cli-<os>-<arch>.tar.gz` (`.zip` on Windows) | The command-line tool `eyv` for Linux, macOS, and Windows on amd64 and arm64, with `agent-crypto`, which agents need for end-to-end encrypted projects, spm, which runs the agent host as a service, and their notices |
| `install.sh` | Installs the command-line tool on Linux or macOS and pairs the machine as an agent host |
| `install.ps1` | The same for Windows, in PowerShell |
| `THIRD_PARTY_NOTICES.txt` | Licenses of what the command-line tool and agent-crypto include |
| `THIRD_PARTY_NOTICES-app.txt` | Licenses of what the apps include |
| `SHA256SUMS`, `SHA256SUMS.sig` | Every file's SHA-256, and its signature |

Releases with a hyphen in the tag, such as `v2.2.0-rc.1`, are prereleases;
`latest` and the install script skip them.

## Command-line tool

On a Linux or macOS machine that runs agents:

```sh
curl -fsSL https://app.eyvant.com/install.sh | sh
```

On Windows, in PowerShell:

```powershell
irm https://app.eyvant.com/install.ps1 | iex
```

Each downloads the archive for the machine, checks it against `SHA256SUMS`,
installs `eyv`, `agent-crypto`, and spm, and pairs the machine as an agent
host: it shows a pairing code, a QR code, and a fingerprint to confirm in the
app. `install.sh` installs in `~/.local/bin`; `install.ps1` in
`%LOCALAPPDATA%\Programs\eyvant`, which it adds to the user's `PATH`. To name
the machine, run `curl -fsSL https://app.eyvant.com/install.sh | sh -s -- -name build-01`,
or on Windows:

```powershell
& ([scriptblock]::Create((irm https://app.eyvant.com/install.ps1))) -name build-01
```

Running the command again on a paired machine updates it to the latest
release. Earlier releases called the command `eyvant`; `install.sh` leaves
`eyvant` as a link to `eyv`.

To install by hand instead, download the archive for the system, unpack it,
and put `eyv` (`eyv.exe` on Windows) on `PATH`, with `agent-crypto` beside it.
On macOS, a file downloaded with a browser is quarantined: run
`xattr -d com.apple.quarantine eyv agent-crypto` before the first run.

## Checking a download

`SHA256SUMS.sig` is the base64 Ed25519 signature of `SHA256SUMS` under this
key:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAFDmO7N29x/WIgZHX11p4NaUm9PySTXkJyZPxlth/2/M=
-----END PUBLIC KEY-----
```

Check it with OpenSSL 3, after saving the key as `release-key.pub`:

```sh
base64 -d SHA256SUMS.sig > SHA256SUMS.sig.bin
openssl pkeyutl -verify -rawin -pubin -inkey release-key.pub -in SHA256SUMS -sigfile SHA256SUMS.sig.bin
sha256sum -c --ignore-missing SHA256SUMS
```

The Android app is signed with the certificate whose SHA-256 is
`21:15:A2:17:F6:06:D2:B6:26:7F:66:0E:4C:3C:7D:E6:33:13:81:AF:AC:B0:3E:61:2C:A1:DA:CC:EA:7D:1D:40`.
`apksigner verify --print-certs eyvant-app-android-arm64-v8a.apk` shows it.
