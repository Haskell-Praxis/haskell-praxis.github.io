---
title: Integrate Travis and Coveralls.io into a Haskell Project
author: Hannes Siebenhandl
---

# Continous Integration

Continous Integration is nothing new.
However, the biggest problems in haskell are around toolchains. How can you generate the documentation?
Can I see somehow the coverage?
What about automatic builds on commit?
Can I integrate my build process into tools like jenkins or
travis?
This blog post aims to show exactly how one can accomplish the same build cycles as everybody is used in languages like java or C.

## Travis

In this blog we are gonna focus on travis.
The main reason is, that it is automatically integrated into github, therefore there is no need for your own server, docker instances, and other setup.
[Only a login via github is required](https://travis-ci.org/).

On the page, one does simply add the repository.
That's all, your program is integrated into Travis.
Ok, that was kind of a lie. There is one more step required.

### Configuration of .travis.yml

Travis needs a Configuration file which specifies how to build your project. Since we want to integrate Haskell, one would expect
to just use the Haskell documentation directly on the [Travis Docs](https://docs.travis-ci.com/user/languages/haskell/).
Unfortunately, this is just a setup for `cabal`. While this is strictly speaking ok, this blog will
explain how to build your project with `stack`. Assuming, your project is currently being built with `stack`.

Personally, I am a lazy person, I do not want to write configuation files from scratch in some
DSL that I have never actually used before. Therefore, I googled a little bit and found almost immediately a great
specification for our `.travis.yml`. The main homepage of [stack](https://docs.haskellstack.org/en) itself provides information
on how to integrate it into stack [\[1\]](https://docs.haskellstack.org/en/stable/travis_ci/).

The project, for which this blog has been written, is a simple project, therefore I only used the simple `.travis.yml` which looks like the following:

```
# This is the simple Travis configuration, which is intended for use
# on applications which do not require cross-platform and
# multiple-GHC-version support. For more information and other
# options, see:
#
# https://docs.haskellstack.org/en/stable/travis_ci/
#
# Copy these contents into the root directory of your Github project in a file
# named .travis.yml

# Use new container infrastructure to enable caching
sudo: false

# Do not choose a language; we provide our own build tools.
language: generic

# Caching so the next build will be fast too.
cache:
  directories:
  - $HOME/.stack

# Ensure necessary system libraries are present
addons:
  apt:
    packages:
      - libgmp-dev

before_install:
# Download and unpack the stack executable
- mkdir -p ~/.local/bin
- export PATH=$HOME/.local/bin:$PATH
- travis_retry curl -L https://www.stackage.org/stack/linux-x86_64 | tar xz --wildcards --strip-components=1 -C ~/.local/bin '*/stack'

install:
# Build dependencies
- stack --no-terminal --install-ghc test --only-dependencies

script:
# Build the package, its tests, and its docs and run the tests
- stack --no-terminal test --haddock --no-haddock-deps

```

The file is pretty self explanatory, except for the caching line. This is needed, since travis would otherwise download GHC everytime it tries to build the project. This would take up to an hour which would be obviously a pain in the ass.

Also interesting are the `install:` and `script:` lines. The first line will be executed during installation process while the script line executes the tests.

Now create the file `.travis.yml` in your project folder and push it to github! After a few minutes, travis will build the project. On the first run, it takes at least ten minutes, so don't worry, you did nothing wrong! Just lean back, grab a coffee, or a club-mate or play with some microcontrollers.

After the build finishes, you will see if the build has succeeded or not. If the build failed, some test have failed, or a compile time error has ocurred. Fix it until you see a success message!

Now every commit will trigger a rebuild and will inform you, about the build process.

However, there are still no coverage reports. What do I know about the quality of my program, if I don't even know, how the coverage is? We will discuss this in the next step.

## Code Coverage with Coveralls.io

Using Coveralls.io isn't as easy to use as Travis.
But first things first, the first step is, to login to [Coveralls.io](coveralls.io) and enable Coverage reports for the right repository.
It should be automatically synced. Coveralls shows documentation on how to configure the project and the `repo_token`. This token is very important as it has to be imported to Travis in order to publish automatic build results. After that, we can start to integrate Coveralls into Travis.

The main problem is that we use stack. There would be a tool for cabal-new-build based projects, however, we use stack.
Actually, there is also a tool for stack, but during writing this code, this project does no compile. To still use this program, I forked it and fixed the problems, it is available on [github](https://github.com/fendor/stack-hpc-coveralls). I added git information to the coveralls json file and removed, for now, the `TRAVIS_BUILD_ID` environment variable. It caused loads of trouble, but I might fix it soon.

Another problem was that travis, for some reason, could not found the supplied binary for shc (name of the binary executable). So, we download it in the build process, move it to `.local/bin`, because this is already on the PATH and it is the typical path for local binaries.

After the build succeeds, we can publish via `shc --repo-token=${repo_token} module-name module-test-name`.
It is important that the following environment variables are defined:

* `repo_token` which can be defined in the travis settings of the project. You can copy it from Coveralls.io

* `TRAVIS` set to `travis-ci`. This can be done either via travis settings or directly in the building script.

* `TRAVIS_BUILD_ID` set to some nonsense value for now. It is required by shc, but since it caused troubles, I removed it from the coverage file.
I did not want to change too much of the program, so I did not remove the need to have this flag. I might change this in the future.

After that, we can define the `.travis.yml`. Without going too much into detail, it now builds shc from source, and after a successful build, coverage data will be published to coveralls.io.

Here is my final `.travis.yml`

```
# This is the simple Travis configuration, which is intended for use
# on applications which do not require cross-platform and
# multiple-GHC-version support. For more information and other
# options, see:
#
# https://docs.haskellstack.org/en/stable/travis_ci/
#
# Copy these contents into the root directory of your Github project in a file
# named .travis.yml

# Use new container infrastructure to enable caching
sudo: false

# Do not choose a language; we provide our own build tools.
language: haskell

# Caching so the next build will be fast too.
cache:
  directories:
  - $HOME/.stack

# Ensure necessary system libraries are present
addons:
  apt:
    packages:
      - libgmp-dev

before_install:
  # Download and unpack the stack executable
  - mkdir -p ~/.local/bin
  - export PATH=$HOME/.local/bin:$PATH
  - export TRAVIS=travis-ci
  - export TRAVIS_JOB_ID=1234 # random number
  - travis_retry curl -L https://www.stackage.org/stack/linux-x86_64 | tar xz --wildcards --strip-components=1 -C ~/.local/bin '*/stack'
  # build shc
  - cd ..
  - git clone https://github.com/fendor/stack-hpc-coveralls.git
  - cd stack-hpc-coveralls
  - stack build
  - cp .stack-work/install/x86_64-linux/lts-8.13/8.0.2/bin/shc ~/.local/bin/
  - cd ../core-catcher
  - ls ../ # debug reasons



install:
  # Build dependencies
  - stack --no-terminal --install-ghc test --only-dependencies

script:
  # Build the package, its tests, and its docs and run the tests
  - stack --no-terminal test :core-catcher-test --coverage --haddock --no-haddock-deps

after_script:
  # publish coverage results coveralls.io
  # after_script might clean some things
  - export TRAVIS_JOB_ID=${TRAVIS_BUILD_ID}
  # publish information
  - ~/.local/bin/shc --repo-token=${repo_token} core-catcher core-catcher-test
  - cat json.file # debug reasons
```
