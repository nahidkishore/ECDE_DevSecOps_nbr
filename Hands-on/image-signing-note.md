
# 🛡️ Container Image Signing Hands-on Lab (Cosign)

### **Objective**

In this hands-on lab, you will:

* ✅ Build a Docker image from a React application
* ✅ Push the image to Docker Hub
* ✅ Sign the image using **Cosign**
* ✅ Verify the image signature

This ensures that your container images are cryptographically signed and tamper-proof, improving supply chain security.

---

## 1️⃣ Pre-Requisites

Before you begin, make sure you have the following:

* ✅ **Docker** installed and running
* ✅ **Cosign (v2.x)** installed
* ✅ A valid **Docker Hub account** with credentials

### Install Cosign

For Linux:

```bash
sudo curl -sSL https://github.com/sigstore/cosign/releases/download/v2.2.0/cosign-linux-amd64 -o /usr/local/bin/cosign
sudo chmod +x /usr/local/bin/cosign
```

For macOS (Homebrew):

```bash
brew install cosign
```

Check installation:

```bash
cosign version
```

Login to Docker Hub:

```bash
docker login
```

---

## 2️⃣ Build the Docker Image

1. Make sure your `Dockerfile` is in the project root.
2. Build the image:

```bash
docker build -t nahid0002/cosign-image-test:v1 .
```

3. Verify the image:

```bash
docker images
```

Expected output:

```
nahid0002/cosign-image-test   v1     <image_id>     <created>     <size>
```

---

## 3️⃣ Push the Image to Docker Hub

Push the image:

```bash
docker push nahid0002/cosign-image-test:v1
```

Confirm upload by visiting:
👉 [https://hub.docker.com/r/nahid0002](https://hub.docker.com/r/nahid0002)

---

## 4️⃣ Generate a Cosign Key Pair

Cosign requires a key pair (private/public) for signing and verifying.

```bash
cosign generate-key-pair
```

You will be prompted for a password:

```
Enter password for private key:
Repeat password for private key:
Private key written to cosign.key
Public key written to cosign.pub
```

* `cosign.key` → **Private key** (keep safe, do not share)
* `cosign.pub` → **Public key** (used for verification, can be shared)

> Best practice: Store keys securely in a **Vault** or **Jenkins Secrets**.

---

## 5️⃣ Sign the Docker Image

Use the private key to sign the pushed image:

```bash
cosign sign --key cosign.key docker.io/nahid0002/cosign-image-test:v1
```

You will be prompted for the password:

```
Enter password for private key:
```

On success, you should see:

```
Successfully signed docker.io/nahid0002/cosign-image-test:v1
```

---

## 6️⃣ Verify the Image Signature

Now verify the signature using the **public key**:

```bash
cosign verify --key cosign.pub docker.io/nahid0002/cosign-image-test:v1
```

Expected output:

```
Verification for docker.io/nahid0002/cosign-image-test:v1 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key
```

✅ This confirms:

* The image has not been tampered with
* The image was signed using your private key

---

# 🎯 Conclusion

You have successfully:

1. Built a Docker image
2. Pushed it to Docker Hub
3. Signed it with **Cosign**
4. Verified the signature

This process ensures **container image integrity** and strengthens your **supply chain security**.

---

