# Discourse Patches of Light

This is a collection of patches for [discourse](https://www.github.com/discourse/discourse) of experimental features and changes required to run certain plugins. Though trying to keep them up with latest development, they might lack latest updated against master. They are provided as is and come with no warranty.

## Details

Patchsets are bundled per directory. Currently these are:

### capability_archetypes

Patches discourse archetype system to be capability driven instead of hardcoded against certain types. Allows extending discourse with new archetypes with still show up and are searchable and alike.

Developed at the `archetype-fixes` branch in https://www.github.com/ligthyear/discourse .

### handlebars_injector

Patches discourse to allow injecting template snippets into the handlebar compiler. Makes extending existing templates much easier and more reliable than using post-rendering-injection.

**Note** at the moment, it doesn't work if handlebar templates are precompiled. Make sure to have `config.handlebars.precompile` set to `false`.

Developed at the `archetype-fixes` branch in https://www.github.com/ligthyear/discourse .

## Apply patches

To apply patches to discourse, run the following commands from your discourse directory:

  git apply PATH_TO_DISCOURSE_PATCHES_CLONE/DIRECTORY/*.patch

## Changelog:

 * 2014-05-05
   - initial version

## Authors:
Benjamin Kampmann <me @ create-build-execute . com>

## License (BSD):
Copyright (c) 2014, Benjamin Kampmann
All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


