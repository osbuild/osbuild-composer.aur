# Maintainer: Achilleas Koutsou <achilleas@koutsou.net>

pkgname=osbuild-composer
pkgdesc='An HTTP service for building bootable OS images'
pkgver=144
pkgrel=1
url="https://www.osbuild.org"
arch=(x86_64)
license=(Apache-2.0)
depends=('dnf' 'qemu' 'osbuild' 'systemd' 'btrfs-progs')
makedepends=('go' 'systemd')
checkdepends=('go')
optdepends=()
source=($pkgname-$pkgver.tar.gz::https://github.com/osbuild/osbuild-composer/archive/refs/tags/v${pkgver}.tar.gz)
sha256sums=('342d2057414d211a449b95de4aa61aaf0ae39d506437a2998a091f32706840f0')

prepare() {
  cd $pkgname-$pkgver
  mkdir -p build

  # Arch doesn't use /usr/libexec: edit service files
  sed -i 's,/usr/libexec,/usr/lib,g' distribution/osbuild-*.service

  # configure path to depsolver
  cat > osbuild-composer.toml << EOF
dnf-json = "/usr/lib/osbuild/osbuild-depsolve-dnf"
EOF
}

build() {
  cd $pkgname-$pkgver
  export LDFLAGS="-ldflags=-X=github.com/osbuild/osbuild-composer/internal/common.RpmVersion=${pkgname}-${pkgver}-${pkgrel}.${arch}"
  export GOFLAGS="-buildmode=pie -trimpath -mod=vendor -modcacherw ${LDFLAGS}"

  go build -o build/osbuild-composer ./cmd/osbuild-composer
  go build -o build/osbuild-worker ./cmd/osbuild-worker
}

package() {
  cd $pkgname-$pkgver

  # binaries
  install -Dm755 "build/osbuild-composer" "${pkgdir}/usr/lib/osbuild-composer/osbuild-composer"
  install -Dm755 "build/osbuild-worker"   "${pkgdir}/usr/lib/osbuild-composer/osbuild-worker"

  # sysusers
  install -Dm644 distribution/osbuild-composer.conf "${pkgdir}/usr/lib/sysusers.d/osbuild-composer.conf"

  # systemd units
  mkdir -p "${pkgdir}/usr/lib/systemd/system"
  install -Dm644 distribution/*.service "${pkgdir}/usr/lib/systemd/system/"
  install -Dm644 distribution/*.socket "${pkgdir}/usr/lib/systemd/system/"

  # repositories
  # mkdir -p "${pkgdir}/usr/share/osbuild-composer/repositories"
  # install -Dm644 repositories/*.json "${pkgdir}/usr/share/osbuild-composer/repositories"

  # config file
  mkdir -p "${pkgdir}/etc/osbuild-composer/"
  install -Dm 644 osbuild-composer.toml "${pkgdir}/etc/osbuild-composer/osbuild-composer.toml"

  # license
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

check() {
  cd $pkgname-$pkgver

  # Some depsolve tests require an initialised rpmdb (at the default location
  # /var/lib/rpm) but we need root to initialise it.
  # Skip depsolve tests that require it if it's not found.
  skips=""
  if ! rpm -qa &> /dev/null; then
    skips="TestErrorRepoInfo|TestDepsolver"
  fi

  export LDFLAGS=""  # clear ldflags for test
  export GOFLAGS="-buildmode=pie -trimpath -mod=vendor -modcacherw ${LDFLAGS}"
  go test -skip "${skips}" ./...
  if [[ ! -z "${skips}" ]]; then
    echo "WARNING: Some tests were skipped because they require an initialised rpmdb"
  fi
}
