# Eyvant releases

Signed builds of the Eyvant apps and command-line tool, built by
`.github/workflows/build.yml` from a release tag of the source.

## Files

| File | What it is |
| --- | --- |
| `eyvant-app-android-arm64-v8a.apk` | The Android app |
| `eyvant-app-windows-x64-setup.exe` | The Windows app's installer. It is not code-signed, so SmartScreen warns on first run |
| `eyvant-cli-<os>-<arch>.tar.gz` (`.zip` on Windows) | The command-line tool `eyv` for Linux, macOS, and Windows on amd64 and arm64, with its notices. The Linux archives also hold `agent-crypto`, which agents need for end-to-end encrypted projects |
| `install.sh` | Installs the command-line tool and pairs the machine as an agent host |
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

It installs `eyv` and, on Linux, `agent-crypto` in `~/.local/bin`, checked
against `SHA256SUMS`. Elsewhere, download the archive for the system, unpack
it, and put `eyv` (`eyv.exe` on Windows) on `PATH`, with `agent-crypto`
beside it on Linux. On macOS, a file downloaded with a browser is quarantined:
run `xattr -d com.apple.quarantine eyv` before the first run. Earlier releases
called the command `eyvant`; the install script leaves `eyvant` as a link to
`eyv`.

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
