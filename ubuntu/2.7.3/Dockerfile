FROM ubuntu:bionic AS bionic-pandoc-haskell

ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_PRIORITY critical

RUN apt-get -q --no-allow-insecure-repositories update \
   && apt-get install --assume-yes --no-install-recommends \
        ca-certificates=20180409 \
        cabal-install=1.24.0.2-2 \
        cpp \
        curl \
        fakeroot \
        gcc \
        ghc \
        libgmp-dev \
        liblua5.3-dev \
        pkg-config \
        make \
        zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

RUN cabal update \
  && cabal install cabal-install \
  && cp "$HOME"/.cabal/bin/cabal /usr/bin/

WORKDIR /app

FROM bionic-pandoc-haskell AS bionic-pandoc-build

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
           # --ghc-options '-O1 -optc-Os -optl=-pthread -fPIC -j +RTS -A128m -n2m -RTS' \
           pandoc pandoc-citeproc
RUN find "${HOME}/.cabal" -name 'pandoc*' -type f -perm -u=x -exec cp '{}' /usr/bin/ ';'

RUN strip /usr/bin/pandoc /usr/bin/pandoc-citeproc


FROM ubuntu:bionic AS bionic-pandoc
ARG pandoc_commit=master
LABEL maintainer='Albert Krewinkel <albert+pandoc@zeitkraut.de>'
LABEL org.pandoc.maintainer='Albert Krewinkel <albert+pandoc@zeitkraut.de>'
LABEL org.pandoc.author "John MacFarlane"
LABEL org.pandoc.version "2.7.3"

COPY --from=bionic-pandoc-build /usr/bin/pandoc* /usr/bin/
RUN apt-get -q --no-allow-insecure-repositories update \
  && apt-get install --assume-yes --no-install-recommends \
       ca-certificates=20180409 \
       liblua5.3-0=5.3.3-1ubuntu0.18.04.1 \
       lua-lpeg=1.0.0-2 \
       libatomic1=8.3.0-6ubuntu1~18.04.1 \
       libgmp10=2:6.1.2+dfsg-2 \
       libpcre3=2:8.39-9 \
       libyaml-0-2=0.1.7-2ubuntu3 \
       zlib1g=1:1.2.11.dfsg-0buntu2 \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /workdir
