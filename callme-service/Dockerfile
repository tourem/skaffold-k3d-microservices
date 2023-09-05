# Custom Java runtime using jlink in a multi-stage container build
FROM eclipse-temurin:17 as jre-build

# Create a custom Java runtime
COPY target/*.jar app.jar

RUN jar xf app.jar

RUN jdeps --ignore-missing-deps -q  \
    --recursive  \
    --multi-release 17  \
    --print-module-deps  \
    --class-path 'BOOT-INF/lib/*'  \
    app.jar > deps.info
RUN jlink \
    --add-modules $(cat deps.info) \
    --strip-debug \
    --compress 2 \
    --no-header-files \
    --no-man-pages \
    --output /javaruntime

FROM debian:stretch-slim as app-build

ENV JAVA_HOME=/opt/java/openjdk
ENV PATH "${JAVA_HOME}/bin:${PATH}"

WORKDIR application

COPY --from=jre-build /javaruntime $JAVA_HOME
COPY target/*.jar application.jar

RUN java -Djarmode=layertools -jar application.jar extract

# Define your base image
FROM debian:stretch-slim
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH "${JAVA_HOME}/bin:${PATH}"

WORKDIR application

COPY --from=jre-build /javaruntime $JAVA_HOME
COPY --from=app-build application/dependencies/ ./
COPY --from=app-build application/spring-boot-loader/ ./
COPY --from=app-build application/snapshot-dependencies/ ./
COPY --from=app-build application/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
