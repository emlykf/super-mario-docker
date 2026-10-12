# Introduction

A Super Mario browser game (TypeScript, HTML5 Canvas), containerized with Docker
and deployed on AWS EC2.

## Challenges & How I Solved Them

### 1. Docker permission denied
- **Problem:** Running `docker build` failed with `permission denied while trying to connect to the Docker daemon socket`.

- **Cause:** My user (`admin`) wasn't allowed to access the Docker daemon. By default, only `root` and members of the `docker` group can.

- **Solution:** Added my user to the `docker` group and started a new SSH session so the change took effect:

  ```bash
  sudo usermod -aG docker $USER
  ```

  Verified with `id admin` and `docker ps`.

### 2. `npm ci` failed during the build
- **Problem:** The build stopped at `RUN npm ci` with an error saying no `package-lock.json` was found.

- **Cause:** Only the Dockerfile was in the build context. The game's source code wasn't on the VM, so `COPY package*.json ./` had nothing to copy.

- **Solution:** Cloned the source code, placed the Dockerfile in the project root, and ran `docker build` from there so the full project was in the build context.

### 3. Game unreachable in the browser
- **Problem:** The container was running and port 8080 was published (`-p 8080:8080`), but the game didn't load at `http://<public-ip>:8080`.

- **Cause:** Cloud providers block inbound traffic by default. The original tutorial used a GCP firewall rule with a network tag; on AWS, the equivalent is a security group rule.

- **Solution:** Added an inbound rule to the instance's security group allowing TCP traffic on port 8080, after which the game loaded immediately.

## 📝 Credit

This project is based on [super-mario-mimic](https://github.com/iam-veeramalla/super-mario-mimic) by Abhishek Veeramalla.