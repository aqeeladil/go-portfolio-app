# Portfolio Website

It is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests. Below is a detailed guide for setting up and executing each step.

## Create a `Dockerfile`

```
# Stage 1: Build the image

# Start with a base image
FROM golang:1.22.5 as base         
# Set the working directory inside the container
WORKDIR /app
# Copy the go.mod and go.sum to the working directory
COPY go.mod ./
# Download all the dependencies
RUN go mod download
# Copy the source code to the working directory
COPY . .
# Build the application
RUN go build -o main .

# Stage 2: Reduce the image size using multi-stage builds

# Use a distroless image to run the application
FROM gcr.io/distroless/base
# Copy the binary from the previous stage
COPY --from=base /app/main .
# Copy the static files from the previous stage
COPY --from=base /app/static ./static
# Expose the port on which the application will run
EXPOSE 8080
# Command to run the application
CMD ["./main"]
```

## Run it locally

```bash
# Update and Install Sysytem Packages
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget apt-transport-https gnupg software-properties-common

# Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
# Log out and log back in to apply the Docker group change.

# Install Go
sudo apt install golang-go

# Clone the repository
git clone https://github.com/aqeeladil/go-portfolio-app.git
cp path/to/Dockerfile path/to/go-portfolio-app/
cd go-portfolio-app

# Build and run
go build -o main .
ls
./main

# Access the application at `http://localhost:8080/courses`
```

