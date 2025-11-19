## Here's a step-by-step guide to configure GitHub SSH key access:

### Step 1: Generate SSH Key on Jenkins Server

- Login to Jenkins server container and switch to jenkins user
```
$ kubectl exec -it -n jenkins            jenkins-75869759b-rg6tc   -- bash
```
- Generate SSH key
```
ssh-keygen -t ed25519 -C "jenkins@example.com"
```
- OR for legacy systems:
```
ssh-keygen -t rsa -b 4096 -C "jenkins@example.com"
```

- Press Enter to save in default location
- Leave passphrase empty for automation purposes

- Display the public key
```
$ cat ~/.ssh/id_ed25519.pub
```
- OR for RSA:
```
cat ~/.ssh/id_rsa.pub
```
- Copy the output - you'll need this for GitHub

### Step 2: Add SSH Key to GitHub

- Go to GitHub → Settings → SSH and GPG keys
- Click "New SSH key"
- Give it a title (e.g., "Jenkins Server")
- Paste the public key you copied
- Click "Add SSH key"

### Step 3: Add SSH Private Key to Jenkins Credentials
- Go to Jenkins → Manage Jenkins → Manage Credentials
- Click on "Jenkins" under Stores scoped to Jenkins
- Click on "Global credentials (unrestricted)"
- Click "Add Credentials"
- Fill in the following:
- Kind: SSH Username with private key
- ID: github-ssh-key (use this ID in your pipeline)
- Description: GitHub SSH Key
- Username: git
- Private Key: Select "Enter directly"
- Copy the contents of ~/.ssh/id_ed25519 (or id_rsa)
- Passphrase: Leave empty if you didn't set one
- Click OK

### Step 4: Test SSH Connection
- Run pipeline to see `git pull` succesful.
