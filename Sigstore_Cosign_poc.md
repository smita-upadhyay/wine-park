# Sigstore Cosign

## Create Jenkins pipeline for Docker Image Signing and Verification with AWS ECR using Cosign within two different jenkins jobs

### Softwares to be installed on EC2

1. JDK
2. MAVEN ( as we are using java app to create an image)
3. Docker
4. Jenkins
5. AWS CLI
6. Sigstore Cosign

### Settings to be done in Jenkins

1. In Global Tool configuration add home path for both JRE and Maven
2. Install plugin `Pipeline: AWS Steps` to store AWS credentils (Access key and access token )
3. Now in Credentials manager add follwing credentials:

   1. Github Token, GITHUB_TOKEN (to access github repository)
   2. cosign password, COSIGN_PASSWORD (to be used to protect private key if generating new key pair)
   3. cosign private key, COSIGN_PRIVATE_KEY ( existing private key provided as a secret file)
   4. cosign public key, COSIGN_PUBLIC_KEY ( existing public key provided as a secret file)
   5. AWS Credentials, AWS_CREDENTIALS (using pipeline: aws steps plugun)

Note: Add jenkins user to the docker group and restart jenkins service to avoid using sudo for docker commands in pipeline

```shell
sudo usermod -aG docker jenkins
```

### By using local key pair

```shell
cosign generate-key-pair
// this command will generate cosign.key and cosign.pub
```

```shell
cosign sign --key $COSIGN_PRIVATE_KEY <ECR URI with tag or digest>
// support for tag will be removed in future updates so we must use digest e.g example.com/user/image@digest....
```

```shell
cosign verify --key $COSIGN_PUBLIC_KEY <ECR URI with tag>
```

### By using key pair from AWS KMS

```shell
cosign generate-key-pair --KMS awskms:///alias/<alias_name>
// this command will generate a key pair and store it on aws KMS 
```

```shell
cosign sign --key awskms:///alias/<alias_name> <ECR URI with tag or digest>
// support for tag will be removed in future updates so we must use digest e.g example.com/user/image@digest....
```

```shell
cosign verify --key awskms:///alias/<alias_name> <ECR URI with tag>
```

We can also export the public key and verify against that file:

```shell
cosign public-key --key awskms:///alias/<alias_name> > kms.pub
cosign verify --key kms.pub <ECR URI with tag>
```

## Create Jenkins pipeline for Docker Image Signing and Verification with GHCR using Cosign


### Softwares to be installed on EC2

1. JDK
2. MAVEN ( as we are using java app to create an image)
3. Docker
4. Jenkins
5. Sigstore Cosign

### Settings to be done in Jenkins

1. In Global Tool configuration add home path for both JRE and Maven
2. Now in Credentials manager add follwing credentials:

   1. Github Token, GITHUB_TOKEN ( username and password)
   2. cosign password, COSIGN_PASSWORD (to be used to protect private key if generating new key pair)
   3. cosign private key, COSIGN_PRIVATE_KEY ( existing private key provided as a secret file)
   4. cosign public key, COSIGN_PUBLIC_KEY ( existing public key provided as a secret file)

Note: Add jenkins user to the docker group and restart jenkins service to avoid using sudo for docker commands in pipeline

```shell
sudo usermod -aG docker jenkins
```

### By using local key pair

```shell
cosign generate-key-pair
// this command will generate cosign.key and cosign.pub
```

```shell
cosign sign --key $COSIGN_PRIVATE_KEY <ECR URI with tag or digest>
// support for tag will be removed in future updates so we must use digest e.g example.com/user/image@digest....
```

```shell
cosign verify --key $COSIGN_PUBLIC_KEY <ECR URI with tag>
```