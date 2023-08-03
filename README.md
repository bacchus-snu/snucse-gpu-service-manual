## SNUCSE GPU Service Manual

## Install mdBook

`cargo`와 GNU `gettext`가 필요하다.
해당 패키지들에 대한 Debian 기준 설치는 다음과 같다.
```sh
curl https://sh.rustup.rs -sSf | sh
sudo apt install gettext
```

mdbook과 i18n-helpers를 설치한다.
```sh
cargo install mdbook
cargo install mdbook-i18n-helpers
```

## Translate in English

mdbook에서 text를 재추출하고 기존 번역에 업데이트를 한다.
```sh
MDBOOK_OUTPUT='{"xgettext": {"pot-file": "messages.pot"}}' \
  mdbook build -d po
msgmerge --update po/en.po po/messages.pot
```
`po/en.po`에서 추가된 문자열을 번역하면 된다.

## Build

```sh
mdbook build  # Korean
MDBOOK_BOOK__LANGUAGE=en mdbook build -d book/en  # English
```

## Serve

```sh
mdbook serve  # Korean
MDBOOK_BOOK__LANGUAGE=en mdbook serve -d book/en  # English
```
