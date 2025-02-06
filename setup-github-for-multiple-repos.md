## Setting Up SSH for GitHub

Before cloning your repository, ensure that your SSH keys are properly set up with GitHub. This allows you to authenticate without entering your username and password every time.

### 1. Generate a New SSH Key (if you don’t have one)

Make sure you choose different name for each, else the first one will be replaced
```sh
ssh-keygen -t ed25519 -C "your_email@anasyd.com"
```

If your system doesn’t support Ed25519, you can use RSA:

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@anasyd.com"
```

Follow the prompts to choose a file location and set a passphrase.

### 2. Add Your SSH Key to the SSH-Agent

Start the SSH agent in the background:

```sh
eval "$(ssh-agent -s)"
```

Then add your SSH key:

```sh
ssh-add ~/.ssh/id_ed25519
```

If you used RSA, replace `id_ed25519` with `id_rsa`.

### 3. Add the SSH Key to Your GitHub Account

Copy the SSH key to your clipboard:

```sh
cat ~/.ssh/id_ed25519.pub
```

Then, follow the instructions on [GitHub's SSH settings page](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) to add your new SSH key.

For more detailed instructions, visit GitHub’s official help page:  
[Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

---

## Configuring SSH for Multiple Repositories

If you are hosting different repositories that require different SSH keys, you can configure your SSH client to use the correct key for each repository by creating (or editing) the `~/.ssh/config` file.

### Example Configuration

Create or open the `~/.ssh/config` file and add entries similar to the following:

```plaintext
# Backend Repository
Host github.com-anasyd-backend
    HostName github.com
    IdentityFile /home/ubuntu/.ssh/ejs_key_ed25519

# Frontend Repository
Host github.com-anasyd-frontend
    HostName github.com
    IdentityFile /home/ubuntu/.ssh/flask_key
```

**How It Works:**

- **Host**: This is an alias you use when connecting. Instead of using `github.com`, you’ll use `github.com-anasyd-backend` or `github.com-anasyd-frontend` in your Git commands.
- **HostName**: This is the actual host name (GitHub).
- **IdentityFile**: This specifies the path to the SSH key you want to use.

### Cloning Using Custom Hostnames

When cloning repositories, use the alias defined in your `~/.ssh/config` file. For example:

- **Backend Repository:**
    
    ```sh
    git clone git@github.com-anasyd-backend:user/backend_repo.git
    ```
    
- **Frontend Repository:**
    
    ```sh
    git clone git@github.com-anasyd-frontend:user/frontend_repo.git
    ```
    
