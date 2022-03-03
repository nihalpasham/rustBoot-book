
![GitHub](https://img.shields.io/github/license/nihalpasham/rustBoot-book) [![ci](https://github.com/nihalpasham/rustBoot-book/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/nihalpasham/rustBoot-book/actions/workflows/ci.yml) [![chat](https://img.shields.io/badge/chat-rustBoot%3Amatrix.org-brightgreen)](https://matrix.to/#/#rustBoot:matrix.org)
# rustBoot Book

This repository contains the source of [`rustBoot book`](https://nihalpasham.github.io/rustBoot-book/index.html).

You can also read the book for free online. 

## Contributing

We'd love your help! Please feel free to submit pull requests.

## Spellchecking

To scan source files for spelling errors, you can use the spellcheck.sh script available in the ci directory. It needs a dictionary of valid words, which is provided in ci/dictionary.txt. If the script produces a false positive (say, you used word BTreeMap which the script considers invalid), you need to add this word to ci/dictionary.txt (keep the sorted order for consistency).