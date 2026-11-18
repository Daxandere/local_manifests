# Local manifest

`amethyst.xml` adds the four repositories needed to build LineageOS 23.2 for
the Redmi Note 14 Pro+ 5G.

The Qualcomm pieces (`device/qcom/sepolicy_vndr/sm8650`,
`hardware/qcom-caf/sm8650`, `hardware/qcom-caf/common` and
`vendor/qcom/opensource/*`) are deliberately absent: LineageOS already provides
them in `snippets/lineage.xml`. Adding them again makes `repo` abort with
`duplicate path`.

## Usage

```bash
mkdir -p .repo/local_manifests
git clone https://github.com/Daxandere/local_manifests .repo/local_manifests
# or: cp amethyst.xml $SRC/.repo/local_manifests/
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

## What it points to

| Path | Repository | Remote |
|---|---|---|
| `device/xiaomi/amethyst` | `Daxandere/android_device_xiaomi_amethyst` | `mine` |
| `device/xiaomi/amethyst-kernel` | `amethyst-playground/android_device_xiaomi_amethyst-kernel` | `upstream` |
| `vendor/xiaomi/amethyst` | `amethyst-playground/proprietary_vendor_xiaomi_amethyst` | `upstream` |
| `hardware/xiaomi` | `amethyst-playground/android_hardware_xiaomi` | `upstream` |

## What it deliberately does not list

LineageOS `snippets/lineage.xml` already brings in the shared Qualcomm trees at
the exact paths this device needs. Re-declaring them is a hard error, not a
warning: `repo` parses every manifest first and aborts the whole sync with
`duplicate path`.

| Path | Provided by LineageOS as |
|---|---|
| `device/qcom/sepolicy_vndr/sm8650` | `android_device_qcom_sepolicy_vndr`, branch `lineage-23.2-caf-sm8650` |
| `device/qcom/sepolicy_vndr/SEPolicy.mk` | `linkfile` from `hardware/qcom-caf/common` |
| `hardware/qcom-caf/sm8650` | audio, display and data-ipa trees |
| `vendor/qcom/opensource/*` | display, power, core-utils-vendor, dataservices, ... |

`core-utils-vendor` is worth noting because it provides the QSPA service
(`vendor.qti.qspa-service` and `qspa_vendor.rc` from `qspaframework/`). The
device tree references those modules but does not define them.

## Switching to your forks

When you fork the vendor or kernel repository, change its `remote` from
`upstream` to `mine` and set the revision to your branch.

Example for the vendor repository:

```xml
<project name="proprietary_vendor_xiaomi_amethyst"
         path="vendor/xiaomi/amethyst"
         remote="mine"
         revision="lineage-23.2" />
```

## HTTPS vs SSH

The manifest uses HTTPS so a fresh build server works without SSH keys. If the
server has an SSH key registered on GitHub, switch the `mine` remote:

```xml
<remote name="mine" fetch="git@github.com:Daxandere" />
```

## Android 17

Do not mix this manifest with `lineage-24.0`. When the 23.2 tree is stable,
create a second manifest with:

- `revision="lineage-24.0"` for the device tree,
- the Android 17 kernel and vendor repositories,
- the same paths.

Keeping both manifests separate avoids accidentally building a 23.2 device
tree against a 24.0 source tree.
