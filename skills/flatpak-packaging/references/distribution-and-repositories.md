# Distribution and Repositories

Related public docs: [Publishing](https://docs.flatpak.org/en/latest/publishing.html), [Repositories](https://docs.flatpak.org/en/latest/repositories.html), [Hosting a repository](https://docs.flatpak.org/en/latest/hosting-a-repository.html), [Single-file bundles](https://docs.flatpak.org/en/latest/single-file-bundles.html), [Flatpak Builder](https://docs.flatpak.org/en/latest/flatpak-builder.html).

## Choose distribution mode

Prefer a Flatpak repository for normal distribution because repositories support updates and deltas. Use single-file bundles for direct downloads, removable media, email attachments, or testing.

Flathub is the common public hosting route. Self-hosted repositories are possible but require signing, HTTP hosting, summary updates, and operational care.

## Export to a repository

```bash
flatpak-builder --force-clean --user --install-deps-from=flathub --repo=repo builddir org.example.App.yml
```

If `repo` does not exist, `flatpak-builder` creates it. Reusing the same `--repo` path adds new commits/versions and can host multiple refs.

Sign repository commits:

```bash
flatpak-builder --gpg-sign=<key-id> --repo=repo builddir org.example.App.yml
flatpak build-update-repo --gpg-sign=<key-id> --generate-static-deltas repo
```

Every commit in a production repository should be signed. Avoid unsigned repositories; if unavoidable for local testing, adding them needs `--no-gpg-verify`.

Generate static deltas to improve download speed:

```bash
flatpak build-update-repo --generate-static-deltas repo
```

Use HTTP keep-alive when hosting because repositories can require many HTTP requests.

## Add and test a local repository

```bash
flatpak remote-add --user --no-gpg-verify local-test repo
flatpak install --user local-test org.example.App
flatpak run org.example.App
flatpak update
```

For signed repos, omit `--no-gpg-verify` and provide the GPG key through repo metadata.

## .flatpakrepo

A `.flatpakrepo` file lets users add a repository:

```ini
[Flatpak Repo]
Title=Example Repo
Url=https://example.org/repo/
Homepage=https://example.org/
Comment=Example Flatpak repository
Description=Example Flatpak repository
Icon=https://example.org/logo.svg
GPGKey=<base64-gpg-key>
```

Generate the base64 GPG key:

```bash
base64 --wrap=0 < key.gpg
```

Users can add it:

```bash
flatpak remote-add --if-not-exists example https://example.org/example.flatpakrepo
```

## .flatpakref

A `.flatpakref` file points to one app in a repository and can install the remote if needed:

```ini
[Flatpak Ref]
Name=org.example.App
Branch=stable
Title=Example App
Url=https://example.org/repo/
RuntimeRepo=https://dl.flathub.org/repo/flathub.flatpakrepo
IsRuntime=false
GPGKey=<base64-gpg-key>
```

Install:

```bash
flatpak install https://example.org/org.example.App.flatpakref
flatpak install org.example.App.flatpakref
```

## Single-file bundles

Create a bundle from a repository:

```bash
flatpak build-bundle repo example.flatpak org.example.App --runtime-repo=https://flathub.org/repo/flathub.flatpakrepo
```

Install directly:

```bash
flatpak install example.flatpak
```

Import into another repository:

```bash
flatpak build-import-bundle other-repo example.flatpak
```

Bundles do not include dependencies or AppStream data. For offline distribution of apps plus dependencies, prefer Flatpak's USB/export workflows over standalone bundles.

## Publishing updates

Adding a new version to a repository makes it available to users. GUI software centers can check automatically; CLI users run:

```bash
flatpak update
```

Repositories store version history efficiently, so clients download only changed data when deltas are available.

## Hosting notes

For static hosting such as GitLab/GitHub Pages:

- Build in CI with a Flatpak-capable image.
- Add Flathub or another runtime remote before building.
- Import/sign GPG keys in CI securely.
- Run `flatpak-builder` with `--repo=repo`.
- Run `flatpak build-update-repo --generate-static-deltas --prune repo`.
- Publish the `repo/` directory as static site content.

Flathub uses `flat-manager`; self-hosting does not have to, but large production repositories benefit from purpose-built tooling.
