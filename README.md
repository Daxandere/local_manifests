# Local manifest

`amethyst.xml` adds the repositories for LineageOS 23.2 on the Redmi Note 14
Pro+ 5G Global. It syncs the device tree, the kernel kit and headers, the vendor
blobs and the matching Xiaomi HALs. The device configuration uses the published
HyperOS Global 307 inputs directly.

The Qualcomm pieces (`device/qcom/sepolicy_vndr/sm8650`,
`hardware/qcom-caf/sm8650`, `hardware/qcom-caf/common` and
`vendor/qcom/opensource/*`) are deliberately absent: LineageOS already provides
them in `snippets/lineage.xml`. Adding them again makes `repo` abort with
`duplicate path`.

## Usage

```bash
mkdir -p ~/android/lineage-23.2
cd ~/android/lineage-23.2
repo init -u https://github.com/LineageOS/android.git -b lineage-23.2 --git-lfs
git clone https://github.com/Daxandere/local_manifests .repo/local_manifests
repo sync -c -j8 --no-clone-bundle --no-tags

unset OUT_DIR OUT_DIR_COMMON_BASE
. build/envsetup.sh
breakfast amethyst
m -j16 bacon
```

Update an existing local-manifest checkout with
`git -C .repo/local_manifests pull --ff-only` before running `repo sync`.

## Branch the manifest syncs

`amethyst.xml` follows the device tree branch `amethyst-23.2-rc1`, the state that
was flashed and tested on the phone: `lineage-23.2` plus the `libperfmgr` power
HAL, the 60-120 Hz boot range, the camera/EIS fixes and the 24 dp shade overlay.
A fresh `repo sync` therefore lands on a tree that builds as-is. Switch that
`revision` back to `lineage-23.2` to build the bare base without the RC work.

The build used for the device is produced with

```bash
TARGET_UNOFFICIAL_BUILD_ID=rcd1 m -j32 bacon
```

which only changes the ZIP name.
For Linux dependencies and build verification, see the
[device build guide](https://github.com/Daxandere/android_device_xiaomi_amethyst/blob/lineage-23.2/docs/BUILD.md).

## What it points to

| Path | Repository | Remote |
|---|---|---|
| `device/xiaomi/amethyst` | `Daxandere/android_device_xiaomi_amethyst` | `mine` |
| `device/xiaomi/amethyst-kernel` | `Daxandere/android_device_xiaomi_amethyst-kernel` | `mine`, branch `lineage-23.2-global307` |
| `vendor/xiaomi/amethyst` | `Daxandere/proprietary_vendor_xiaomi_amethyst` | `mine`, branch `lineage-23.2` |
| `hardware/xiaomi` | `amethyst-playground/android_hardware_xiaomi` | `upstream`, tested revision pinned in XML |

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
