FROM amazoncorretto:17.0.7-alpine

# Add app user
ARG APPLICATION_USER=appuser
RUN adduser --no-create-home -u 1000 -D $APPLICATION_USER

# Configure working directory
RUN mkdir /app && \
    chown -R $APPLICATION_USER /app

USER 1000

COPY --chown=1000:1000 ./target/payroll-0.0.1-SNAPSHOT.jar /app/app.jar
WORKDIR /app

ENTRYPOINT [ "java", "-jar", "/app/app.jar" ]
EXPOSE 8080

