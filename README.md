# vector_vendor

A ROS 2 `ament_cmake` package that fetches a checksum-pinned build of
[Vector](https://vector.dev/) — a single static binary observability data pipeline — from
its official release tarball at build time.

It exists to vendor Vector for the [DC (`ros2_data_collection`)](https://github.com/Minipada/ros2_data_collection)
pipeline, where Vector is the external **Shipper** the Bridge (`dc_bridge`) supervises at
runtime. See that repo's [ADR-0002](https://github.com/Minipada/ros2_data_collection/blob/jazzy/docs/adr/0002-vector-as-default-shipper.md)
for why Vector was chosen, and its amendments for why this package fetches the release
tarball live via `file(DOWNLOAD ...)` rather than checking a binary into git history:
ROS buildfarm binarydeb jobs do have network access at build time, so there's no reason
to pay the "every version bump grows git history by ~106MB, permanently" cost of a
checked-in binary.

## Why a separate repo

Vendor packages `ros2_data_collection` pulls in via `.repos` live in their own repos as a
rule, for a consistent layout and bump/release workflow across all of them — see that
repo's ADR-0012.

## Usage

Pull this repo into a ROS 2 workspace alongside `ros2_data_collection` (e.g. via a
`.repos` file and `vcs import`), then `colcon build` as usual. Network access is
required at build time to fetch the pinned release tarball.

```sh
colcon build --packages-select vector_vendor
```

### Bumping the vendored Vector version

1. Update `VECTOR_VERSION` and the per-architecture `VECTOR_SHA256_*` checksums in
   `CMakeLists.txt` to match the new release
   (find them on [vectordotdev/vector releases](https://github.com/vectordotdev/vector/releases)).
2. Update `package.xml`'s `<version>` to match `VECTOR_VERSION` — this package's own
   version tracks the vendored Vector version exactly, not an independent release
   number, so the two must always agree.
3. Tag the commit `v<VECTOR_VERSION>` (e.g. `v0.58.0`) and push the tag — consumers
   (`ros2_data_collection`'s `.repos` file) pin to a tag, not a floating branch, for
   reproducible builds.

### Using a system-installed Vector instead

Set `vector_path` (a colcon `--cmake-args -Dvector_path=/usr/bin/vector`) or the
`VECTOR_PATH` environment variable to skip the download entirely — useful for
air-gapped builds or distro-packaged Vector installs.

## License

MPL-2.0 for this package's own files (`CMakeLists.txt`, `package.xml`). The Vector
binary this package fetches and installs at build time is MPL-2.0, © 2020 Vector
Authors &lt;vector@datadoghq.com&gt;. See [LICENSE.md](./LICENSE.md).
