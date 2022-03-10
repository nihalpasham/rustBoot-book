
![GitHub](https://img.shields.io/github/license/nihalpasham/rustBoot-book) ![GitHub Workflow Status (event)](https://img.shields.io/github/workflow/status/nihalpasham/rustBoot-book/ci) [![chat](https://img.shields.io/badge/chat-rustBoot%3Amatrix.org-brightgreen)](https://matrix.to/#/#rustBoot:matrix.org)
# rustBoot Book

This repository contains the source for the `rustBoot book`.

You can read the book for [`free online`](https://nihalpasham.github.io/rustBoot-book/index.html).

## Contributing

We'd love your help! Please feel free to submit pull requests.

## Spellchecking

To scan source files for spelling errors, you can use the spellcheck.sh script available in the ci directory. It needs a dictionary of valid words, which is provided in ci/dictionary.txt. If the script produces a false positive (say, you used word BTreeMap which the script considers invalid), you need to add this word to ci/dictionary.txt (keep the sorted order for consistency).