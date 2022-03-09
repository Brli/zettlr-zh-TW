# Maintainer: BrLi <brli at chakralinux dot org>
# Maintainer: Caleb Maclennan <caleb@alerque.com>

pkgname=zettlr-tw
pkgver=2.2.4
pkgrel=1
pkgdesc="A markdown editor for writing academic texts and taking notes"
arch=('x86_64')
url='https://www.zettlr.com'
license=('GPL' 'custom') # Noted that the icon and name are copyrighted
depends=(electron pandoc texlive-bin ttf-lato)
makedepends=(git yarn nodejs-lts-gallium) # check .github/workflows/build.yml for NODE_VERSION
_csl_locale_commit=c38205618f1a23eb80e8c5f33c8086648ca3874b # Dec 23, 2021
_csl_style_commit=ccb71844fdafb2b7a48cccb364f4b4c03d3cdce6  # Sep 19, 2021
provides=(zettlr)
conflicts=(zettlr)
options=(!strip)
source=(git+https://github.com/Zettlr/Zettlr#tag=v${pkgver}
        git+https://github.com/Brli/zettlr-zh-TW.git
        0002-Do-not-download-pandoc.patch
        # citation style
        "locales-${pkgver}-${pkgrel}.zip::https://github.com/citation-style-language/locales/archive/${_csl_locale_commit}.zip"
        "chicago-author-date-${pkgver}-${pkgrel}.csl::https://github.com/citation-style-language/styles/raw/${_csl_style_commit}/chicago-author-date.csl")
sha256sums=('SKIP'
            'SKIP'
            '97d1e620226324a1b7e5571ca800286a62f17e3729b08383918be81b64530287'
            '275fc80a391b4002b52182deb12997a1408118b7753977ea69ef5256c6f3ff47'
            '1455e57b314fd13ba155f4ab93f061e3e6393c13cd0f16380adb9d73614f7930')

prepare() {
    # zh_TW patch
    cd "${srcdir}/Zettlr/static"
    patch -Np3 -i "${srcdir}/zettlr-zh-TW/zh_TW.patch"
    cp ${srcdir}/zettlr-zh-TW/zh-TW.json lang/

    cd "${srcdir}/Zettlr"
    patch -Np1 -i $srcdir/0002-Do-not-download-pandoc.patch
    ln -sf /usr/bin/pandoc resources/pandoc-linux-x64
    ln -sf /usr/bin/pandoc resources/pandoc

    # csl:refresh from package.json
    find "${srcdir}/locales-$_csl_locale_commit" -name "*.xml" \
        -exec cp {} static/csl-locales/ \;
    cp "${srcdir}/locales-$_csl_locale_commit/locales.json" static/csl-locales/
    cp "${srcdir}/chicago-author-date-${pkgver}-$pkgrel.csl" static/csl-styles/chicago-author-date.csl
}

build() {
    cd "${srcdir}/Zettlr"
    local NODE_ENV=''
    yarn install --cache-folder "${srcdir}/cache" \
                 --link-folder "${srcdir}/link"
    yarn reveal:build

    node node_modules/.bin/electron-forge package

    cd "${srcdir}/Zettlr/.webpack"
    # Remove references to $srcdir
    find renderer -type f -name 'index.js.map' -exec sed -i "s,${srcdir}/Zettlr,/usr/lib/${pkgname},g" {} +

    # Remove resources for other OSs and leftovers
    cd "${srcdir}/Zettlr/resources"
    rm -rfv NSIS
    rm -rfv icons/dmg
    rm -rfv icons/*icns
    rm -rfv icons/*.ico
    rm -rfv screenshots
    rm -r .gitignore
}

# check() {
#     cd "${srcdir}/Zettlr"
#     # Require electron module to test
#     yarn add --cache-folder "${srcdir}/cache" --link-folder "${srcdir}/link" electron
#     # The "test" function in package.json
#     node node_modules/mocha/bin/mocha
#     # The "test-gui" function in package.json, not useful in our case
#     node scripts/test-gui.js
#     # Clean up
#     yarn remove electron
#     rm yarn.lock
#     rm node_modules/.bin -rf
# }

package() {
    local _destdir=usr/lib/"${pkgname}"
    install -dm755 "${pkgdir}/${_destdir}"

    cd "${srcdir}/Zettlr"

    # Copy the generated electron project
    cp -r --no-preserve=ownership --preserve=mode ./.webpack "${pkgdir}/${_destdir}/"
    cp -r --no-preserve=ownership --preserve=mode ./resources "${pkgdir}/${_destdir}/"
    cp -r --no-preserve=ownership --preserve=mode ./static "${pkgdir}/${_destdir}/"
    cp -r --no-preserve=ownership --preserve=mode ./package.json "${pkgdir}/${_destdir}/"

    # Install start script to /usr/bin
    install -Dm755 /dev/stdin "${pkgdir}/usr/bin/Zettlr" <<END
#!/bin/sh
exec electron /${_destdir} "\$@"
END

    # install icons of various sizes to hi-color theme
    for px in 16 24 32 48 64 96 128 256 512 1024; do
        install -Dm644 "${srcdir}/Zettlr/resources/icons/png/${px}x${px}.png" \
            "${pkgdir}/usr/share/icons/hicolor/${px}x${px}/apps/Zettlr.png"
    done

    # Generate desktop entry file
    # Based on descriptions in package.json and forge.config.js
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/applications/Zettlr.desktop" <<END
[Desktop Entry]
Name=Zettlr
Comment=A powerful Markdown Editor with integrated tree view
Exec=Zettlr %U
Terminal=false
Type=Application
Icon=Zettlr
StartupWMClass=Zettlr
MimeType=text/markdown;
Categories=Office;
END

    # Generate mimetype configuration file to associate with zettlr
    # Based on electron-builder.yml
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/mime/packages/Zettlr.xml" <<END
<?xml version="1.0" encoding="utf-8"?>
<mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
    <mime-type type="text/markdown">
        <glob pattern="*.md"/>
            <comment>Markdown document</comment>
        <icon name="x-office-document" />
    </mime-type>
    <mime-type type="text/markdown">
        <glob pattern="*.markdown"/>
            <comment>Markdown document</comment>
        <icon name="x-office-document" />
    </mime-type>
    <mime-type type="text/markdown">
        <glob pattern="*.rmd"/>
            <comment>RMarkdown document</comment>
        <icon name="x-office-document" />
    </mime-type>
</mime-info>
END

    # license
    install -Dm644 -t "${pkgdir}/usr/share/licenses/${pkgname}/" "${srcdir}/Zettlr/LICENSE"
}
