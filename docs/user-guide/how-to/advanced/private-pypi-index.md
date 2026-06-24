# Host a Private PyPI Index|
| `resolve_deps: false` | Downloads only the exact requirements you list (`pip download --no-deps`). You must list every transitive dependency yourself. | Tightest control. Each artifact is audited individually. |
| `resolve_deps: true` | Downloads each requirement plus its transitive dependencies, as resolved by pip. | Less manual work, but you trust pip's choice of transitive versions. |

Pin versions in either case. An unpinned entry resolves to whatever is latest at
build time, which defeats the point of a curated index.

Restrict the index to the Python version your resources actually run. Saturn
Cloud's default Python images run a specific interpreter version (check with
`python --version` in a resource). Building for `3.11` produces `cp311` wheels
that will not install on other interpreters.

## 1. The build job

Create the build script and config as a [Resource Recipe](<docs/user-guide/how-to/advanced/recipes.md>).
The recipe ships both files through `config_files`, which Saturn Cloud writes to
`$HOME` (`/home/jovyan`) at pod startup. Each key is a path relative to `$HOME`;
the value holds the file `content` and an octal `mode`. The job needs the shared
folder mounted with write access.

```yaml
type: job
spec:
  name: pypi-index-builder
  owner: your-org/your-identity
  image: saturncloud/saturn-python:2025.05.01
  instance_type: r6alarge
  description: Download curated packages and regenerate the static PEP 503 index.
  working_directory: /home/jovyan
  command: python /home/jovyan/build_index.py --config /home/jovyan/packages.yaml --root /home/jovyan/shared/pypi-index/pypi
  extra_packages:
    pip:
      install: dumb-pypi pyyaml
  environment_variables:
    PYPI_ROOT: /home/jovyan/shared/pypi-index/pypi
  shared_folders:
    - name: pypi-index
      owner: your-org/your-identity
      path: /home/jovyan/shared/pypi-index
  config_files:
    packages.yaml:
      mode: "0644"
      content: |
        resolve_deps: false
        python_versions:
          - "3.11"
        platforms: []
        packages:
          - "requests==2.32.3"
          - "urllib3==2.2.3"
          - "certifi==2024.8.30"
          - "charset-normalizer==3.4.0"
          - "idna==3.10"
    build_index.py:
      mode: "0644"
      content: |
        import argparse, os, shutil, subprocess, sys
        from pathlib import Path
        import yaml

        def run(cmd):
            print("+", " ".join(cmd), flush=True)
            subprocess.run(cmd, check=True)

        def download(config, packages_dir):
            packages_dir.mkdir(parents=True, exist_ok=True)
            resolve_deps = bool(config.get("resolve_deps", False))
            py_versions = config.get("python_versions") or []
            platforms = config.get("platforms") or []
            reqs = config.get("packages") or []
            if not reqs:
                sys.exit("packages.yaml has no packages entries")
            base = [sys.executable, "-m", "pip", "download", "--dest", str(packages_dir)]
            if not resolve_deps:
                base.append("--no-deps")
            if py_versions or platforms:
                base.append("--only-binary=:all:")
            for v in py_versions:
                base += ["--python-version", v]
            for p in platforms:
                base += ["--platform", p]
            for req in reqs:
                run(base + [req])

        def generate_index(packages_dir, web_dir):
            if web_dir.exists():
                shutil.rmtree(web_dir)
            web_dir.mkdir(parents=True, exist_ok=True)
            filenames = sorted(
                p.name for p in packages_dir.iterdir()
                if p.suffix in (".whl", ".gz", ".zip") or p.name.endswith(".tar.gz")
            )
            if not filenames:
                sys.exit("no distribution files found in %s" % packages_dir)
            listing = packages_dir.parent / "_filelist.txt"
            listing.write_text("\n".join(filenames) + "\n")
            print("%d artifacts" % len(filenames), flush=True)
            run([
                sys.executable, "-m", "dumb_pypi.main",
                "--package-list", str(listing),
                "--packages-url", "../../packages/",
                "--output-dir", str(web_dir),
            ])
            # Symlink packages/ into web/ so a single document root resolves the
            # relative hrefs, with no web-server alias config required.
            link = web_dir / "packages"
            if link.is_symlink() or link.exists():
                link.unlink()
            link.symlink_to("../packages")
            print("index written to %s/simple/" % web_dir, flush=True)

        def main():
            ap = argparse.ArgumentParser()
            ap.add_argument("--config", default="packages.yaml")
            ap.add_argument("--root", default=os.environ.get("PYPI_ROOT", "/home/jovyan/shared/pypi"))
            args = ap.parse_args()
            config = yaml.safe_load(Path(args.config).read_text())
            root = Path(args.root)
            download(config, root / "packages")
            generate_index(root / "packages", root / "web")
            print("DONE", flush=True)

        if __name__ == "__main__":
            main()
```

Run the job once. The logs show each package downloading and the index being
written. To change the package set later, edit the `packages:` list, re-apply
the recipe, and run the job again. The index is regenerated from scratch each
run, so removed packages disappear from it.

## 2. The Caddy deployment

Serve the static tree with [Caddy](https://caddyserver.com/). Caddy is a single
static binary, so it runs as the non-root user in a Saturn Cloud container with
no image changes. The deployment below fetches the binary onto the shared folder
on first start (subsequent starts reuse it) and serves the index.

Do not use `python -m http.server` for this. It handles one request at a time,
which serializes every `pip install` and stalls all clients behind a single
large wheel download. Caddy serves requests concurrently with `sendfile` and
connection timeouts.

```yaml
type: deployment
spec:
  name: pypi-index
  owner: your-org/your-identity
  image: saturncloud/saturn-python:2025.05.01
  instance_type: r6alarge
  scale: 1
  description: Curated PyPI index (PEP 503), static files over Caddy.
  environment_variables:
    PYPI_ROOT: /home/jovyan/shared/pypi-index/pypi
  shared_folders:
    - name: pypi-index
      owner: your-org/your-identity
      path: /home/jovyan/shared/pypi-index
  routes:
    - container_port: 8000
      visibility: unauthenticated
  command: >-
    bash -lc '
    CADDY=/home/jovyan/shared/pypi-index/caddy;
    if [ ! -x "$CADDY" ]; then
      curl -fsSL "https://caddyserver.com/api/download?os=linux&arch=amd64" -o "$CADDY";
      chmod +x "$CADDY";
    fi;
    exec "$CADDY" file-server --root "$PYPI_ROOT/web" --listen 0.0.0.0:8000 --browse
    '
```

A Saturn Cloud deployment proxy always targets container port `8000`, and the
server must bind `0.0.0.0`, both set above.

## Route visibility and pip

The `unauthenticated` route visibility is required for `pip`. The other
visibility levels (`org`, `account`, `owner`) put the deployment behind an
interactive OAuth proxy. `pip` cannot complete that browser login flow, so it
follows the redirect to a login page, finds no package links, and reports
`Could not find a version that satisfies the requirement`. With an
unauthenticated route, access is controlled by network reachability: only
clients that can reach the cluster network can read the index. Combine that with
blocking public PyPI at the network layer.

If you change a deployment's route visibility, stop and start the deployment.
The proxy renders its configuration at startup, so re-applying the recipe alone
leaves the previous visibility in effect.

## Use the index

Point `pip` at the deployment's URL with the `/simple/` path:

```bash
pip install -i https://<deployment-url>/simple/ requests
```

To make it the default for a resource, set it once:

```bash
pip config set global.index-url https://<deployment-url>/simple/
```

Or set `PIP_INDEX_URL` as an [Environment Variable](/docs)
on the resource so every install uses the curated index without a flag.

## Notes

- The build job is the only component that needs access to public PyPI. You can
  run it from a network segment that has PyPI access and keep the serving
  segment with no outbound PyPI route at all. The two only share the folder.
- The index serves wheels built for the Python version set in `packages.yaml`.
  Add more versions to `python_versions` if you support multiple interpreters.
- The deployment serves files per request, so a rebuild by the job is picked up
  with no deployment restart.
