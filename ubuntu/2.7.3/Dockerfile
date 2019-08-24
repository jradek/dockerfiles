FROM ubuntu:bionic AS builder

ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_PRIORITY critical

WORKDIR /app

RUN apt-get -q --no-allow-insecure-repositories update \
  && apt-get install --assume-yes --no-install-recommends \
       software-properties-common \
  && add-apt-repository ppa:hvr/ghc \
  && apt-get install --assume-yes --no-install-recommends \
        ghc-8.6.5 cabal-install-2.4 \
        build-essential=12.4* \
        ca-certificates=* \
        curl=7.58.* \
        fakeroot=1.22-* \
        libgmp-dev=2:6.1.* \
        liblua5.3-dev=5.3.3* \
        pkg-config=0.29.1* \
        zlib1g-dev=1:1.2.11.* \
  && rm -rf /var/lib/apt/lists/* \
  && ln -s /opt/ghc/bin/ghc-8.6.5 /usr/bin/ghc \
  && ln -s /opt/cabal/bin/cabal-2.4 /usr/bin/cabal

# NOTE: --ghc-options -j +RTS -A128m -n2m -RTS, see:
# https://rybczak.net/2016/03/26/how-to-reduce-compilation-times-of-haskell-projects/
RUN cabal --version \
  && ghc --version \
  && cabal new-update

RUN cabal new-install \
           --flag embed_data_files \
           --flag bibutils \
           --constraint 'hslua +system-lua +pkg-config +hardcode-reg-keys' \
           --symlink-bindir="${HOME}/.cabal/bin" \
           --disable-debug-info --enable-executable-stripping \
           # --ghc-options '-O1 -optc-Os -optl=-pthread -fPIC -j +RTS -A128m -n2m -RTS' \
           pandoc-2.7.3 pandoc-citeproc pandoc-crossref
RUN find "${HOME}/.cabal" -name 'pandoc*' -type f -perm -u=x -exec cp '{}' /usr/bin/ ';'

# Cabal's exec stripping doesn't seem to work reliably, so let's do it again.
RUN strip /usr/bin/pandoc /usr/bin/pandoc-citeproc


FROM ubuntu:bionic AS bionic-pandoc
ARG pandoc_commit=master
LABEL maintainer='Albert Krewinkel <albert+pandoc@zeitkraut.de>'
LABEL org.pandoc.maintainer='Albert Krewinkel <albert+pandoc@zeitkraut.de>'
LABEL org.pandoc.author "John MacFarlane"
LABEL org.pandoc.version "2.7.3"

ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_PRIORITY critical

COPY --from=builder /usr/bin/pandoc* /usr/bin/
RUN apt-get -q --no-allow-insecure-repositories update \
  && apt-get install --assume-yes --no-install-recommends \
       ca-certificates=* \
       liblua5.3-0=5.3.3* \
       lua-lpeg=1.0.* \
       libatomic1=8.3.* \
       libgmp10=2:6.1.* \
       libpcre3=2:8.39-* \
       libyaml-0-2=0.1.* \
       zlib1g=1:1.2.11.* \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /workdir
