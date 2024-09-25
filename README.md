---

# 🍽️ React Restaurant Website - Azure Deployment Guide

This guide provides a step-by-step process to deploy the React Restaurant Website on an **Azure Virtual Machine (VM)** using Docker. You'll learn how to set up the Azure VM, install Docker, and run the application.

---

## 🚀 Quick Overview

- **Tech Stack**: React, Node.js, NGINX
- **Deployment**: Docker on Azure VM
- **Repository**: [React Restaurant Website Theme](https://github.com/walifile/react-resturant-website-theme.git)

---

## 📋 Prerequisites

Before you begin, ensure you have the following:

- An **Azure account** with permissions to create resources.
- **Azure Virtual Machine** with at least **8 GB RAM** (we'll cover the setup).
- Basic knowledge of Azure and terminal commands.

---

## 🛠️ Step-by-Step Guide

### Step 1: Set Up an Azure Virtual Machine

#### 1.1 Create a New Virtual Machine

1. **Log in to the Azure Portal**:  
   Go to [portal.azure.com](https://portal.azure.com) and sign in to your account.

2. **Navigate to Virtual Machines**:  
   In the left sidebar, click on **"Virtual machines"**, then click **"Create"** > **"Azure virtual machine"**.

3. **Configure Basic Settings**:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Create a new resource group or use an existing one.
   - **Virtual Machine Name**: Choose a name (e.g., `react-restaurant-vm`).
   - **Region**: Select a region close to you.
   - **Image**: Choose **Ubuntu Server 20.04 LTS**.
   - **Size**: Select a VM size with at least **8 GB RAM** (e.g., **Standard DS2 v2**).

4. **Set Administrator Account**:
   - **Authentication Type**: Choose **SSH public key**.
   - **Username**: Enter your desired username.
   - **SSH public key source**: Use existing public key
   - **SSH Public Key**: Paste your public SSH key.

5. **Configure Inbound Port Rules**:
   - **Public Inbound Ports**: Select **Allow selected ports**.
   - **Select Inbound Ports**: Choose **SSH (22)** and **HTTP (80)**.

6. **Review and Create**:
   - Click **"Review + create"**.
   - Review the settings and click **"Create"**.

#### 1.2 Obtain the Public IP Address

- After deployment, navigate to your VM and note the **Public IP address**. You'll need this to connect via SSH and to access the website.

---

### Step 2: Connect to Your Azure VM via SSH

1. **Open Terminal**: On your local machine.

2. **Connect to the VM**:
   ```bash
   ssh <username>@<public_ip_address>
   ```
   Replace `<username>` with your VM's username and `<public_ip_address>` with your VM's public IP.

---

### Step 3: Install Docker on the Azure VM

1. **Update System Packages**:
   ```bash
   sudo apt update
   ```

2. **Install Docker**:
   ```bash
   sudo apt install -y docker.io
   ```

3. **Change the owner for docker**:
   ```bash
   sudo chown $USER /var/run/docker.sock
   ```

4. **Verify Docker Installation**:
   ```bash
   docker --version
   ```

---

### Step 4: Clone the Repository on the VM

1. **Clone the Project**:
   ```bash
   git clone https://github.com/walifile/react-resturant-website-theme.git
   ```

2. **Navigate into the Project Directory**:
   ```bash
   cd react-resturant-website-theme
   ```

---

### Step 5: Create the Dockerfile on the VM

1. **Create a Dockerfile**:
   ```bash
   nano Dockerfile
   ```

2. **Paste the Following Code** into the `Dockerfile`:

   ```Dockerfile
   # Use an official Node runtime as a base image for building the app
   FROM node:18-alpine AS builder

   # Set the working directory in the container
   WORKDIR /usr/src/app

   # Copy package.json and package-lock.json to the container
   COPY package*.json ./

   # Install frontend dependencies
   RUN npm install

   # Update browserslist database
   RUN npx update-browserslist-db@latest

   # Set environment variable to enable legacy OpenSSL support
   ENV NODE_OPTIONS=--openssl-legacy-provider

   # Copy the rest of the application files to the container
   COPY . .

   # Build the frontend app
   RUN npm run build

   # Use NGINX as a lightweight base image to serve the app
   FROM nginx:alpine

   # Copy the built app from the previous stage
   COPY --from=builder /usr/src/app/build /usr/share/nginx/html

   # Expose port 80 to the outside world (default for HTTP)
   EXPOSE 80

   # Start NGINX server
   CMD ["nginx", "-g", "daemon off;"]
   ```

3. **Save the Dockerfile** and exit (`Ctrl + s`, then `Ctrl + x` to exit).

---

### Step 6: Build the Docker Image on the VM

1. **Build the Docker Image**:
   ```bash
   docker build -t react-restaurant-site .
   ```
   This command builds the Docker image and tags it as `react-restaurant-site`.

---

### Step 7: Run the Docker Container on the VM

1. **Run the Container**:
   ```bash
   docker run -d -p 80:80 react-restaurant-site
   ```
   - `-d`: Runs the container in detached mode.
   - `-p 80:80`: Maps port `80` of the VM to port `80` of the container.

2. **Verify the Container is Running**:
   ```bash
   docker ps
   ```
   You should see `react-restaurant-site` listed.

---

### Step 8: Configure Azure Network Security Group (if needed)

If you didn't open port `80` during VM creation, you need to allow inbound traffic on port `80`.

1. **Navigate to Your VM** in the Azure Portal.

2. **Click on "Networking"** in the sidebar.

3. **Add Inbound Port Rule**:
   - **Destination port ranges**: `80`
   - **Protocol**: `TCP`
   - **Action**: `Allow`
   - **Priority**: A number (e.g., `1000`)
   - **Name**: `Allow-HTTP`

4. **Save** the rule.

---

### Step 9: Access the Deployed Website

1. **Open Your Web Browser**.

2. **Navigate to**:
   ```
   http://<public_ip_address>
   ```
   Replace `<public_ip_address>` with your VM's public IP.

3. **You Should See the React Restaurant Website!**

---

## 📂 Additional Docker Commands

- **List Running Containers**:
   ```bash
   docker ps
   ```

- **Stop a Running Container**:
   ```bash
   docker stop <container_id>
   ```

- **Remove a Container**:
   ```bash
   docker rm <container_id>
   ```

- **View Docker Images**:
   ```bash
   docker images
   ```

---

## 🛠️ Troubleshooting

### Common Issues

- **Port Already in Use**:  
  If you encounter an error that port `80` is already in use, try using a different port:
  ```bash
  docker run -d -p 8080:80 react-restaurant-site
  ```
  Then, ensure that port `8080` is allowed in the Azure Network Security Group and access the site via `http://<public_ip_address>:8080`.

- **Permission Denied Errors**:  
  Ensure your user is added to the Docker group:
  ```bash
  sudo usermod -aG docker $USER
  ```

  Then, log out and log back in.
  Or
  ```bash
  sudo chown $USER /var/run/docker.sock
  ```


- **Docker Build Fails**:  
  Ensure all dependencies are correctly installed and that you're connected to the internet.

---

## 🔒 Security Considerations

- **Firewall Rules**:  
  Opening ports to the internet can expose your VM to security risks. Ensure you only open necessary ports and consider using Azure Firewall or Network Security Groups for added security.

- **Updates**:  
  Regularly update your system packages and Docker to the latest versions.

---

## 📞 Support

If you encounter any issues or have questions, please open an issue in this repository or contact the project maintainer.

---

## 🙌 Conclusion

Congratulations! You have successfully deployed the React Restaurant Website on an Azure Virtual Machine using Docker. This deployment setup is scalable and can be integrated with Azure services like Azure Container Instances or Azure Kubernetes Service for more advanced scenarios.

---

