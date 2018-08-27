#!/bin/sh
# Copyright Cédric Picard 2014 -- License WTFPL
# Inspired by Ranger's -- See https://github.com/hut/ranger

set -e -u

if [ $# -ne 0 ] ; then
    { for i in "$@" ; do printf "%s\n" "$i" ; done } | "$0"
    exit $?
fi

EDITOR="${EDITOR:-vi}"

namebase="$(mktemp -d)/blkrn"
echo '# Modify filenames without removing any line, quitting aborts' \
    > "${namebase}.2"
tee -- "${namebase}.1" >> "${namebase}.2"
exec </dev/tty >/dev/tty \
    || { echo 'Interactive terminal needed' >&2 ; exit 1; }

"$EDITOR" "${namebase}.2"
sed -i -- '1d' "${namebase}.2"

if [ $(wc -l < "${namebase}.1") -ne $(wc -l < "${namebase}.2") ] ; then
    rm -r -- "$(dirname -- "${namebase}")"
    echo "Wrong number of lines" >&2 ; exit 1
fi

(echo '# Please review/modify this script or empty it to do nothing' \
    ; while read -r l1 <&3 && read -r l2 <&4; do
        [ "$l1" = "$l2" ] || printf "%s\n%s\n" "$l1" "$l2"
      done 3<"${namebase}.1" 4<"${namebase}.2" \
    | sed 's/\([\\"$`]\)/\\\1/g;s/^.*$/"&"/' \
    | xargs -d"\n" -L2 echo 'mv -i --') \
    > "${namebase}.sh"

if [ "$(wc -c < "${namebase}.sh")" -ne 0 ] ; then
    "$EDITOR" -- "${namebase}.sh"
    sh -e -- "${namebase}.sh"
fi

rm -r -- "$(dirname "${namebase}")"
