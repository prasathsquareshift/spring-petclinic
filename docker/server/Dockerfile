#Multi-Stage build

#Build application stage
#We need maven.
FROM maven:3.8-openjdk-17
ARG JAVA_AGENT_BRANCH=main
ARG JAVA_AGENT_REPO=elastic/apm-agent-java
ARG JAVA_AGENT_BUILT_VERSION=1.44.1-SNAPSHOT

WORKDIR /usr/src/java-code
#build the application
RUN git clone https://github.com/elastic/spring-petclinic.git
WORKDIR /usr/src/java-code/spring-petclinic
RUN mvn -q -B package -DskipTests

RUN mkdir /usr/src/java-app
RUN cp -v /usr/src/java-code/spring-petclinic/target/*.jar /usr/src/java-app/app.jar

#build the agent
WORKDIR /usr/src/java-agent-code
#RUN curl -L https://github.com/$JAVA_AGENT_REPO/archive/$JAVA_AGENT_BRANCH.tar.gz | tar --strip-components=1 -xz
RUN git clone https://github.com/elastic/apm-agent-java.git
WORKDIR /usr/src/java-agent-code/apm-agent-java
RUN mvn clean install -DskipTests=true -Dmaven.javadoc.skip=true


#RUN export JAVA_AGENT_BUILT_VERSION=$(mvn -q -Dexec.executable="echo" -Dexec.args='${project.version}' --non-recursive org.codehaus.mojo:exec-maven-plugin:1.3.1:exec) \
#    && cp -v /usr/src/java-agent-code/apm-agent-java/elastic-apm-agent/target/elastic-apm-agent-${JAVA_AGENT_BUILT_VERSION}.jar /usr/src/java-app/elastic-apm-agent.jar
RUN cp -v /usr/src/java-agent-code/apm-agent-java/elastic-apm-agent/target/elastic-apm-agent-${JAVA_AGENT_BUILT_VERSION}.jar /usr/src/java-app/elastic-apm-agent.jar

FROM openjdk:17

RUN export
WORKDIR /app

COPY --from=0 /usr/src/java-app/*.jar ./

CMD java -javaagent:/app/elastic-apm-agent.jar\
                                        -Dspring.profiles.active=${JAVA_PROFILE:-hsqldb,spring-data-jpa}\
                                        -Dserver.port=${SERVER_PORT:-}\
                                        -Delastic.apm.application_packages=org.springframework.samples.petclinic\
                                        -Dserver.context-path=/petclinic/\
                                        -Dspring.messages.basename=messages/messages\
                                        -Dlogging.level.org.springframework=${LOG_LEVEL:-INFO}\
                                        -Dsecurity.ignored=${SECURITY_IGNORED:-/**}\
                                        -Dspring.datasource.initialize=${INITIALIZE_DB:-false}\
                                        -Dbasic.authentication.enabled=${AUTHENTICATION_ENABLED:-false}\
                                        -Dserver.address=${SERVER_ADDRESS:-0.0.0.0}\
                                        -Dspring.datasource.url=${DATABASE_URL:-jdbc:hsqldb:mem:petclinic}\
                                        -Dspring.datasource.username=${DATABASE_USERNAME:-sa}\
                                        -Dspring.datasource.password=${DATABASE_PASSWORD:-}\
                                        -Dspring.datasource.driver-class-name=${DATABASE_DRIVER:-}\
                                        -Dspring.jpa.database=${DATABASE_DIALECT:-HSQL}\
                                        -Dspring.jpa.database-platform=${DATABASE_PLATFORM:-org.hibernate.dialect.HSQLDialect}\
                                        -Dspring.jpa.hibernate.ddl-auto=${DDL_AUTO:-none}\
                                        -Dspring.datasource.schema=${DATASOURCE_SCHEMA:-classpath*:db/hsqldb/initDB.sql}\
                                        -Dspring.datasource.data=${DATASOURCE_DATA:-classpath*:db/hsqldb/populateDB.sql}\