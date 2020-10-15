FROM opensuse/tumbleweed AS builder

ENV CARDANO_NODE_VERSION="master"

RUN zypper -n install libsodium-devel libopenssl-devel zlib-devel gcc-c++ gcc ghc \
cabal-install git-core ghc-aeson-devel ghc-directory-devel ghc-bytestring-devel \
ghc-containers-devel ghc-deepseq-devel ghc-text-devel ghc-time-devel ghc-exceptions-devel \
ghc-process-devel ghc-network-devel ghc-async-devel ghc-unix-devel ghc-StateVar-devel \
ghc-clock-devel ghc-fail-devel ghc-safe-devel ghc-psqueues-devel ghc-parser-combinators-devel \
ghc-split-devel ghc-byteorder-devel ghc-aeson-pretty-devel ghc-syb-devel ghc-extra-devel \
ghc-ansi-terminal-devel ghc-zlib-devel ghc-ghc-devel ghc-cryptonite-devel ghc-conduit-devel \
ghc-free-devel ghc-either-devel ghc-yaml-devel


WORKDIR /usr/src
RUN git clone --recurse-submodules https://github.com/input-output-hk/cardano-node
WORKDIR /usr/src/cardano-node
RUN git fetch --all
RUN git checkout ${CARDANO_NODE_VERSION}
RUN git cherry-pick -n 60500f976d2e43d59f6cb890972dc2b6845b1f6d # Add the upcoming Allegra and Mary eras to CardanoBlock #1958
RUN git cherry-pick -n ed51dbc465018de90ee99a9c2fdefa1ad832d08d # Add the upcoming Allegra and Mary eras to CardanoBlock #1958
RUN git cherry-pick -n 34cf5f3105831e4a5f0c8407a0e1c6f0e408c43f # Add the upcoming Allegra and Mary eras to CardanoBlock #1958
RUN git cherry-pick -n 7d1a72803625c7c57d25d695aa418b13b2a3a84d # Add the upcoming Allegra and Mary eras to CardanoBlock #1958
RUN scripts/gen-cabal-nosystemd.sh

# Ugly workaround for vty-5.25.1
# RUN ln -s /usr/lib64/libtinfo.so.6 /usr/lib64/libtinfo.so
RUN sed -i '1d' cabal.project # try without fixed index state
RUN cabal v2-update
RUN cabal v2-install cardano-cli \
--allow-newer=base \
--ghc-options="-Wno-error=unused-imports" \
--project-file=cabal.nosystemd.project \
--installdir=/usr/local/bin \
--install-method=copy \
-f -external-libsodium-vrf \
-j
RUN cabal v2-install cardano-node \
--allow-newer=base \
--ghc-options="-Wno-error=unused-imports" \
--project-file=cabal.nosystemd.project \
--installdir=/usr/local/bin \
--install-method=copy \
-f -external-libsodium-vrf \
-j 

FROM opensuse/tumbleweed

LABEL maintainer="Mark Stopka <mark.stopka@perlur.cloud>"

ENV SERVICE_NAME "cardano-node"

EXPOSE 9000

COPY --from=builder /usr/local/bin/cardano-node /usr/local/bin/
COPY --from=builder /usr/local/bin/cardano-cli /usr/local/bin/
COPY usr/local/lib/* /usr/local/lib/
COPY usr/local/bin/* /usr/local/bin/

RUN zypper -n in shadow util-linux

RUN mkdir -p /srv/cardano/

RUN /usr/sbin/useradd -c "Cardano node user" \
            -d /srv/cardano/cardano-node/ \
            -m \
            -r \
            -s /sbin/nologin \
            -U \
            cardano-node

RUN mkdir -p /var/run/cardano/
RUN chown cardano-node.cardano-node /var/run/cardano/
USER cardano-node

RUN mkdir /srv/cardano/cardano-node/{etc,log,storage}/
RUN mkdir /srv/cardano/cardano-node/etc/secrets/

WORKDIR /srv/cardano/cardano-node/

COPY srv/cardano/cardano-node/ /srv/cardano/cardano-node/

HEALTHCHECK --interval=1s --timeout=2s CMD cnode-ctl healthcheck || exit 1

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
