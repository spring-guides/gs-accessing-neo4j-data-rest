FROM adoptopenjdk/openjdk8:latest

RUN apt-get update && apt-get install -y wget unzip

RUN apt-get clean \
 && rm -rf /var/lib/apt/lists/*

ARG NEO4J_VERSION=3.1.1

RUN wget dist.neo4j.org/neo4j-community-${NEO4J_VERSION}-unix.tar.gz
RUN tar -xzf neo4j-community-${NEO4J_VERSION}-unix.tar.gz
RUN mv neo4j-community-${NEO4J_VERSION} neo4j-community-installed