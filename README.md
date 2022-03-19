# MOSIP Configuration
# Build Docker image
docker build -t kumusoft/kernel-config-server:testing .
# Run Docker image
docker run -it -p 51000:51000 kumusoft/kernel-config-server:testing
## License
This project is licensed under the terms of [Mozilla Public License 2.0](LICENSE).
