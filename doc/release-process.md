Release Process
====================

Before every release candidate:

* Update translations.
* Update manpages, see [gen-manpages.sh](/contrib/devtools/README.md#gen-manpagessh).

Before every minor and major release:

* Update [bips.md](bips.md) to account for changes since the last release.
* Update version in `configure.ac` (don't forget to set `CLIENT_VERSION_IS_RELEASE` to `true`).
* Write release notes (see `doc/release-notes/` and `doc/release-notes.md`).
* Update `src/chainparams.cpp` `nMinimumChainWork` with information from the `getblockchaininfo` RPC.
* Update `src/chainparams.cpp` `defaultAssumeValid` with information from the `getblockhash` RPC.
  - The selected value must not be orphaned so it may be useful to set the value two blocks back from the tip.
  - Testnet should be set some tens of thousands back from the tip due to reorgs there.
  - This update should be reviewed with a `reindex-chainstate` with `assumevalid=0` to catch any defect that causes rejection of blocks in the past history.

Before every major release:

* Update hardcoded [seeds](/contrib/seeds/README.md).
* Update [`BLOCK_CHAIN_SIZE`](/src/qt/intro.cpp) to the current size plus some overhead.
* Update `src/chainparams.cpp` `chainTxData` with statistics about the transaction count and rate. Use the output of the RPC `getchaintxstats`. Reviewers can verify the results by running `getchaintxstats <window_block_count> <window_last_block_hash>`.
* Update version of `contrib/gitian-descriptors/*.yml`.

### First time / New builders

If you're using the automated script (found in [contrib/gitian-build.py](/contrib/gitian-build.py)), then at this point you should run it with the `--setup` command. Otherwise ignore this.

Check out the source code in the following directory hierarchy:

    cd /path/to/your/toplevel/build
    git clone https://github.com/LIMXTEC/gitian.sigs.git
    git clone https://github.com/LIMXTEC/megacoin-detached-sigs.git
    git clone https://github.com/devrandom/gitian-builder.git
    git clone https://github.com/LIMXTEC/Megacoin.git megacoin

### Megacoin maintainers/release engineers, suggestion for writing release notes

Write release notes. `git shortlog` helps a lot, for example:

    git shortlog --no-merges v(current version)..v(new version)

Generate list of authors:

    git log --format='- %aN' v(current version)..v(new version) | sort -fiu

Tag version (or release candidate) in git:

    git tag -s v(new version)

### Setup and perform Gitian builds

If you're using the automated script (found in [contrib/gitian-build.py](/contrib/gitian-build.py)), then at this point you should run it with the `--build` command. Otherwise ignore this.

Setup Gitian descriptors:

    pushd ./megacoin
    export SIGNER="(your Gitian key)"
    export VERSION=(new version, e.g. 1.10.0)
    git fetch
    git checkout v${VERSION}
    popd

Ensure your `gitian.sigs` are up-to-date if you wish to gverify your builds against other Gitian signatures:

    pushd ./gitian.sigs
    git pull
    popd

Ensure `gitian-builder` is up-to-date:

    pushd ./gitian-builder
    git pull
    popd

### Fetch and create inputs: (first time, or when dependency versions change)

    pushd ./gitian-builder
    mkdir -p inputs
    wget -P inputs https://bitcoincore.org/cfields/osslsigncode-Backports-to-1.7.1.patch
    wget -P inputs http://downloads.sourceforge.net/project/osslsigncode/osslsigncode/osslsigncode-1.7.1.tar.gz
    popd

Create the macOS SDK tarball, see the [macOS readme](README_osx.md) for details, and copy it into the inputs directory.

### Optional: Seed the Gitian sources cache and offline git repositories

By default, Gitian will fetch source files as needed. To cache them ahead of time, make sure you have checked out the tag you want to build in Megacoin, then:

    pushd ./gitian-builder
    make -C ../megacoin/depends download SOURCES_PATH=`pwd`/cache/common
    popd

Offline builds must use the `--url` flag:

    pushd ./gitian-builder
    ./bin/gbuild --url megacoin=/path/to/megacoin,signature=/path/to/sigs {rest of arguments}
    popd

### Build and sign Megacoin Core for Linux, Windows, and macOS:

    pushd ./gitian-builder
    ./bin/gbuild --num-make 2 --memory 3000 --commit megacoin=v${VERSION} ../megacoin/contrib/gitian-descriptors/gitian-linux.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-linux --destination ../gitian.sigs/ ../megacoin/contrib/gitian-descriptors/gitian-linux.yml
    mv build/out/megacoin-*.tar.gz build/out/src/megacoin-*.tar.gz ../

    ./bin/gbuild --num-make 2 --memory 3000 --commit megacoin=v${VERSION} ../megacoin/contrib/gitian-descriptors/gitian-win.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-win-unsigned --destination ../gitian.sigs/ ../megacoin/contrib/gitian-descriptors/gitian-win.yml
    mv build/out/megacoin-*-win-unsigned.tar.gz inputs/megacoin-win-unsigned.tar.gz
    mv build/out/megacoin-*.zip build/out/megacoin-*.exe ../

    ./bin/gbuild --num-make 2 --memory 3000 --commit megacoin=v${VERSION} ../megacoin/contrib/gitian-descriptors/gitian-osx.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-unsigned --destination ../gitian.sigs/ ../megacoin/contrib/gitian-descriptors/gitian-osx.yml
    mv build/out/megacoin-*-osx-unsigned.tar.gz inputs/megacoin-osx-unsigned.tar.gz
    mv build/out/megacoin-*.tar.gz build/out/megacoin-*.dmg ../
    popd

Build output expected:

  1. source tarball (`megacoin-${VERSION}.tar.gz`)
  2. linux 32-bit and 64-bit dist tarballs (`megacoin-${VERSION}-linux[32|64].tar.gz`)
  3. windows 32-bit and 64-bit unsigned installers and dist zips (`megacoin-${VERSION}-win[32|64]-setup-unsigned.exe`, `megacoin-${VERSION}-win[32|64].zip`)
  4. macOS unsigned installer and dist tarball (`megacoin-${VERSION}-osx-unsigned.dmg`, `megacoin-${VERSION}-osx64.tar.gz`)
  5. Gitian signatures (in `gitian.sigs/${VERSION}-<linux|{win,osx}-unsigned>/(your Gitian key)/`)

### Verify other gitian builders signatures to your own (Optional)

Verify signatures:

    pushd ./gitian-builder
    ./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-linux ../megacoin/contrib/gitian-descriptors/gitian-linux.yml
    ./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-win-unsigned ../megacoin/contrib/gitian-descriptors/gitian-win.yml
    ./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-osx-unsigned ../megacoin/contrib/gitian-descriptors/gitian-osx.yml
    popd

### Next steps:

Commit your signature to `gitian.sigs`:

    pushd gitian.sigs
    git add ${VERSION}-linux/"${SIGNER}"
    git add ${VERSION}-win-unsigned/"${SIGNER}"
    git add ${VERSION}-osx-unsigned/"${SIGNER}"
    git commit -m "Add ${VERSION} unsigned sigs for ${SIGNER}"
    git push
    popd

Codesigner only: Create Windows/macOS detached signatures:

Codesigner only: Sign the macOS binary:

    transfer megacoin-osx-unsigned.tar.gz to macOS for signing
    tar xf megacoin-osx-unsigned.tar.gz
    ./detached-sig-create.sh -s "Key ID"
    Move signature-osx.tar.gz back to the gitian host

Codesigner only: Sign the windows binaries:

    tar xf megacoin-win-unsigned.tar.gz
    ./detached-sig-create.sh -key /path/to/codesign.key
    signature-win.tar.gz will be created

Codesigner only: Commit the detached codesign payloads:

    cd ~/megacoin-detached-sigs
    git add -a
    git commit -m "point to ${VERSION}"
    git tag -s v${VERSION} HEAD
    git push the current branch and new tag

Non-codesigners: wait for Windows/macOS detached signatures:

- Detached signatures will then be committed to the [megacoin-detached-sigs](https://github.com/LIMXTEC/megacoin-detached-sigs) repository.

Create (and optionally verify) the signed macOS binary:

    pushd ./gitian-builder
    ./bin/gbuild -i --commit signature=v${VERSION} ../megacoin/contrib/gitian-descriptors/gitian-osx-signer.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-signed --destination ../gitian.sigs/ ../megacoin/contrib/gitian-descriptors/gitian-osx-signer.yml
    ./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-osx-signed ../megacoin/contrib/gitian-descriptors/gitian-osx-signer.yml
    mv build/out/megacoin-osx-signed.dmg ../megacoin-${VERSION}-osx.dmg
    popd

Create (and optionally verify) the signed Windows binaries:

    pushd ./gitian-builder
    ./bin/gbuild -i --commit signature=v${VERSION} ../megacoin/contrib/gitian-descriptors/gitian-win-signer.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-win-signed --destination ../gitian.sigs/ ../megacoin/contrib/gitian-descriptors/gitian-win-signer.yml
    ./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-win-signed ../megacoin/contrib/gitian-descriptors/gitian-win-signer.yml
    mv build/out/megacoin-*win64-setup.exe ../megacoin-${VERSION}-win64-setup.exe
    mv build/out/megacoin-*win32-setup.exe ../megacoin-${VERSION}-win32-setup.exe
    popd

Commit your signature for the signed macOS/Windows binaries:

    pushd gitian.sigs
    git add ${VERSION}-osx-signed/"${SIGNER}"
    git add ${VERSION}-win-signed/"${SIGNER}"
    git commit -a
    git push
    popd

### After 3 or more people have gitian-built and their results match:

- Create `SHA256SUMS.asc` for the builds, and GPG-sign it:

```bash
sha256sum * > SHA256SUMS
```

The list of files should be:
```
megacoin-${VERSION}-aarch64-linux-gnu.tar.gz
megacoin-${VERSION}-arm-linux-gnueabihf.tar.gz
megacoin-${VERSION}-i686-pc-linux-gnu.tar.gz
megacoin-${VERSION}-x86_64-linux-gnu.tar.gz
megacoin-${VERSION}-osx64.tar.gz
megacoin-${VERSION}-osx.dmg
megacoin-${VERSION}.tar.gz
megacoin-${VERSION}-win32-setup.exe
megacoin-${VERSION}-win32.zip
megacoin-${VERSION}-win64-setup.exe
megacoin-${VERSION}-win64.zip
```

- GPG-sign it, delete the unsigned file:
```bash
gpg --digest-algo sha256 --clearsign SHA256SUMS # outputs SHA256SUMS.asc
rm SHA256SUMS
```

- Announce the release:

  - Upload release artifacts and SHA256SUMS.asc to the official Megacoin channels / megacoin-mec.cc.

  - Archive release notes for the new version to `doc/release-notes/` (branch `master` and branch of the release).

  - Create a [new GitHub release](https://github.com/LIMXTEC/Megacoin/releases/new) with a link to the archived release notes.
