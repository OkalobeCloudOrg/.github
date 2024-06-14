To set up an SSH key connection with GitHub, follow these detailed steps:

### Step 1: Generate an SSH Key
If you don't have an SSH key yet, you'll need to generate one.

1. **Open a terminal**.
2. **Generate a new SSH key** using the following command:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```
   If you're using an older version of SSH that doesn't support `ed25519`, use `rsa`:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

3. **Follow the instructions**. When prompted to enter a location to save the key, press Enter to accept the default location. You can also specify a custom path if you prefer.

4. **Enter a passphrase** to secure your SSH key (recommended but optional).

### Step 2: Add the SSH Key to the SSH Agent
1. **Start the SSH agent**:
   ```bash
   eval "$(ssh-agent -s)"
   ```

2. **Add your private key to the SSH agent**:
   ```bash
   ssh-add ~/.ssh/id_ed25519
   ```
   Replace `id_ed25519` with your key file name if you chose a different name.

### Step 3: Add the SSH Key to Your GitHub Account
1. **Copy the contents of your public key** to the clipboard:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   Use `pbcopy` on macOS:
   ```bash
   pbcopy < ~/.ssh/id_ed25519.pub
   ```
   Or `clip` on Windows:
   ```bash
   clip < ~/.ssh/id_ed25519.pub
   ```

2. **Log in to your GitHub account**.
3. **Navigate to your SSH settings**:
   - Click on your profile picture in the top right corner of the page.
   - Select **Settings**.
   - In the left sidebar, click on **SSH and GPG keys**.
   - Click on **New SSH key**.

4. **Paste your public key** into the "Key" field and give it a descriptive title, then click **Add SSH key**.

### Step 4: Test the SSH Connection
1. **Test your SSH connection to GitHub**:
   ```bash
   ssh -T git@github.com
   ```

2. If this is your first time connecting to GitHub with this key, you may be asked to verify GitHub's fingerprint. Type `yes` to continue.

If everything is set up correctly, you should see a welcome message from GitHub indicating that authentication was successful.

### Conclusion
You have now configured an SSH connection with GitHub. You can use this method to clone repositories, push commits, and more without having to enter your GitHub credentials each time.

14/06/2024
