Integrating SonarCloud with your Dockerfile involves setting up the SonarCloud scanner to analyze your project's source code and then push the results to SonarCloud. This typically involves a few steps:

1. **Installing SonarScanner**: SonarScanner is the tool that will analyze your codebase and send results to SonarCloud.
2. **Configuring the Project**: You'd need a `sonar-project.properties` file to specify details about your project.
3. **Running SonarScanner**: Execute the scanner to analyze your codebase.

Here's how you can modify your Dockerfile to integrate SonarCloud:

1. Install SonarScanner:

```Dockerfile
# Install SonarScanner (modify the version if needed)
ENV SONAR_SCANNER_VERSION=4.4.0.2170
RUN apt-get update && apt-get install -y wget unzip && \
    wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}-linux.zip -O /tmp/sonar-scanner.zip && \
    unzip /tmp/sonar-scanner.zip -d /opt && \
    rm /tmp/sonar-scanner.zip && \
    ln -s /opt/sonar-scanner-${SONAR_SCANNER_VERSION}-linux/bin/sonar-scanner /usr/local/bin/sonar-scanner
```

2. Ensure you have a `sonar-project.properties` file in your project directory. This file contains configurations for SonarCloud. An example file might look like:

```
sonar.projectKey=my_project_key
sonar.organization=my_organization_key
sonar.sources=.
sonar.host.url=https://sonarcloud.io
sonar.login=my_token
```

Replace `my_project_key`, `my_organization_key`, and `my_token` with appropriate values from your SonarCloud account.

3. Execute SonarScanner to analyze your code and send results to SonarCloud:

```Dockerfile
RUN sonar-scanner
```
Note:

- Ensure the `sonar-project.properties` file is copied into the Docker image (using the `COPY` command) before running the `sonar-scanner`.
- You might need to manage sensitive data like the `sonar.login` token securely, especially if this Dockerfile is intended for public use or is stored in a public repository. Consider using Docker secrets or other mechanisms to inject secrets at runtime.
- Depending on your CI/CD pipeline, you might want to separate the SonarCloud analysis from the Docker build process, as building the Docker image and analyzing the code are conceptually two different steps. If you're using tools like Jenkins, GitHub Actions, etc., they offer plugins or actions to run SonarCloud analysis separately.