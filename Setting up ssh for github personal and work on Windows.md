# Setting Up Personal and Work SSH Keys on Windows for GitHub

This guide provides step-by-step instructions for creating separate SSH keys for your personal and work GitHub accounts on Windows, adding them to the respective accounts, and cloning repositories. This setup allows you to manage multiple GitHub accounts without conflicts by using SSH configuration.

## Prerequisites
- Install Git for Windows (includes Git Bash): Download from [git-scm.com](https://git-scm.com/downloads) if not already installed.
- Ensure OpenSSH is installed (included in Windows 10 version 1809+ and Windows 11). If missing:
  1. Go to **Settings > Apps > Optional features**.
  2. Click **Add a feature** and select **OpenSSH Client**.
- Have two GitHub accounts: one personal (e.g., associated with `personal@example.com`) and one work (e.g., `work@example.com`).
- Use Git Bash for most commands (search for "Git Bash" in the Start menu). Avoid running as administrator unless specified.

## Step 1: Generate SSH Key Pairs
You'll create two separate key pairs using `ssh-keygen`. Use Ed25519 for better security if supported; otherwise, fall back to RSA.

### Generate Personal SSH Key
1. Open Git Bash.
2. Run the command, replacing `personal@example.com` with your personal GitHub email:
   ```
   ssh-keygen -t ed25519 -C "personal@example.com"
   ```
   - If Ed25519 isn't supported, use:
     ```
     ssh-keygen -t rsa -b 4096 -C "personal@example.com"
     ```
3. When prompted for the file location, enter a custom name like `~/.ssh/id_ed25519_personal` (or `~/.ssh/id_rsa_personal` for RSA). This avoids overwriting defaults.
4. Enter a secure passphrase (recommended for security) and confirm it. Press Enter twice for no passphrase.
5. The keys will be saved in `C:\Users\YourUsername\.ssh\` (e.g., `id_ed25519_personal` for private key and `id_ed25519_personal.pub` for public key).

### Generate Work SSH Key
1. In Git Bash, run the command, replacing `work@example.com` with your work GitHub email:
   ```
   ssh-keygen -t ed25519 -C "work@example.com"
   ```
   - Or for RSA:
     ```
     ssh-keygen -t rsa -b 4096 -C "work@example.com"
     ```
2. Enter a custom file name like `~/.ssh/id_ed25519_work`.
3. Enter and confirm a passphrase (or skip).
4. Keys will be saved similarly in your `.ssh` folder.

## Step 2: Add Private Keys to the SSH Agent
The SSH agent manages your keys and passphrases. On Windows, start the service first.

1. Open an **elevated PowerShell** (right-click PowerShell in Start menu and select "Run as administrator").
2. Start the SSH agent service:
   ```
   Get-Service -Name ssh-agent | Set-Service -StartupType Manual
   Start-Service ssh-agent
   ```
3. Switch back to a **non-elevated Git Bash** window.
4. Add your personal private key:
   ```
   ssh-add ~/.ssh/id_ed25519_personal
   ```
   - Enter your passphrase if prompted.
5. Add your work private key:
   ```
   ssh-add ~/.ssh/id_ed25519_work
   ```
   - Enter passphrase if applicable.
6. Verify keys are loaded:
   ```
   ssh-add -l
   ```
   - This lists your added keys. If needed, clear existing keys first with `ssh-add -D`.

**Note:** Restarting your computer stops the agent; repeat steps 1-5 to restart and re-add keys.

## Step 3: Configure SSH for Multiple Keys
Create a config file to map keys to specific GitHub "hosts" (aliases for github.com).

1. In Git Bash, navigate to your SSH directory:
   ```
   cd ~/.ssh/
   ```
2. Create or edit the `config` file (use `touch config` if it doesn't exist, then open in a text editor like Notepad or VS Code):
   ```
   notepad config
   ```
3. Add the following content (adjust file names if using RSA or different names):
   ```
   # Personal GitHub account
   Host github.com-personal
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519_personal

   # Work GitHub account
   Host github.com-work
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519_work
   ```
4. Save and close the file.
5. Set permissions (optional but recommended for security):
   ```
   chmod 600 config
   ```

## Step 4: Add Public Keys to GitHub Accounts
Add each public key to its respective GitHub account.

### Add Personal Public Key
1. Copy the personal public key to your clipboard:
   - In Git Bash: `clip < ~/.ssh/id_ed25519_personal.pub`
   - Or open the `.pub` file in a text editor and copy its contents (starts with `ssh-ed25519` or `ssh-rsa`).
2. Log in to your **personal GitHub account** at github.com.
3. Click your profile picture > **Settings**.
4. In the sidebar, under "Access," click **SSH and GPG keys**.
5. Click **New SSH key**.
6. Enter a title (e.g., "Personal Windows Machine").
7. Select **Authentication key** as the type.
8. Paste the public key into the "Key" field.
9. Click **Add SSH key** (confirm if prompted).

### Add Work Public Key
1. Copy the work public key: `clip < ~/.ssh/id_ed25519_work.pub`.
2. Log in to your **work GitHub account**.
3. Repeat steps 3-9 above, using a title like "Work Windows Machine."

## Step 5: Test SSH Connections
Verify each key works.

1. In Git Bash, test personal:
   ```
   ssh -T git@github.com-personal
   ```
   - If prompted about host authenticity, verify the fingerprint matches GitHub's (see [GitHub's SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)) and type `yes`.
   - Expected response: "Hi your_personal_username! You've successfully authenticated..."
2. Test work:
   ```
   ssh -T git@github.com-work
   ```
   - Same verification process.
3. If errors occur (e.g., "Permission denied"), check troubleshooting at [GitHub Docs](https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey).

## Step 6: Clone Repositories
Use the custom hosts from your SSH config to clone repos with the correct key.

1. For a personal repo (replace `your_username/repo` with actual):
   ```
   git clone git@github.com-personal:your_username/repo.git
   ```
2. For a work repo:
   ```
   git clone git@github.com-work:your_work_username/repo.git
   ```
3. Navigate into the cloned repo and set local Git config (to associate commits with the correct account):
   - Personal:
     ```
     cd repo
     git config user.name "Your Personal Name"
     git config user.email "personal@example.com"
     ```
   - Work:
     ```
     git config user.name "Your Work Name"
     git config user.email "work@example.com"
     ```
4. Perform Git operations as usual (e.g., `git add .`, `git commit -m "Message"`, `git push`).

**Tips:**
- If cloning fails, ensure the repo URL uses the alias (e.g., `github.com-personal` instead of `github.com`).
- For existing local repos, update the remote URL: `git remote set-url origin git@github.com-personal:your_username/repo.git`.
- Secure your passphrases and keys; back up your `.ssh` folder if needed.

This setup ensures seamless switching between personal and work GitHub interactions on the same Windows machine.