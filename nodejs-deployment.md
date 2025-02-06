This guide includes setting up your nodeJS app on a server using pm2 and configuring Nginx as a reverse proxy using both the traditional `sites-available`/`sites-enabled` method and the `conf.d` method.

---

# Setup Guide for Node.js/Express.js Environment

Before starting, update and upgrade your packages:

```sh
sudo apt update
sudo apt upgrade
```

---

## Installing Node.js and npm

**What are they?**

- **Node.js**: A JavaScript runtime that lets you run JavaScript outside the browser. It’s required for development tools like Metro (used by React Native).
- **npm**: The Node Package Manager, bundled with Node.js, is used to install and manage JavaScript packages.

You have two options for installation:

### Option A: Install via NodeSource Repository

1. **Add the Node.js Repository and Install:**
    
    ```sh
    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```
    
2. **Verify Installation:**
    
    ```sh
    node --version
    npm --version
    ```
    
    You should see version numbers for both Node.js and npm.
    

---

### Option B: Install via nvm (Node Version Manager)

**Why use nvm?**  
`nvm` allows you to manage multiple Node.js versions on the same machine, making it easy to switch between versions required by different projects.

1. **Install nvm:**
    
    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    ```
    
    > **Note:** Check the [nvm GitHub repo](https://github.com/nvm-sh/nvm) for the latest version if needed.
    
2. **Load nvm into Your Shell:**
    
    Either close and reopen your terminal or run:
    
    ```sh
    source ~/.nvm/nvm.sh
    ```
    
3. **Install a Specific Node.js Version (e.g., v18):**
    
    ```sh
    nvm install 18
    ```
    
4. **Set the Default Node.js Version:**
    
    ```sh
    nvm alias default 18
    ```
    
5. **Verify Installation:**
    
    ```sh
    node --version
    npm --version
    ```
    
6. **Switching Between Versions:**  
    To install and switch to Node.js v16 later, run:
    
    ```sh
    nvm install 16
    nvm use 16
    ```
    
    Switch back to v18 with:
    
    ```sh
    nvm use 18
    ```
    

---

## Setting Up SSH for GitHub

Before cloning your repository, ensure that your SSH keys are properly set up with GitHub. This allows you to authenticate without entering your username and password every time.

### 1. Generate a New SSH Key (if you don’t have one)

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
[Read Here](https://github.com/anasyd/Server-Deployment/blob/main/setup-github-for-multiple-repos.md).

---

## Cloning Your Repository

For a single repository, clone using Git:

```sh
git clone git@github.com:user/repo_name.git
```

If you are using a custom host from your SSH config for multiple repositories, use the appropriate alias.

Then navigate into the repository and install dependencies:

```sh
cd repo_name
npm install
```

---

## Setting Up PM2 to Manage Your Application

PM2 is a process manager for Node.js applications. Install it globally using npm:

```sh
npm install -g pm2
```

### Start Your Application with PM2

Start your app (for example, `index.js`) and assign it a name:

```sh
pm2 start index.js --name your_app
```

Save the PM2 process list:

```sh
pm2 save
```

### Enable PM2 Startup on Boot

Generate the startup script with:

```sh
pm2 startup
```

You’ll see output similar to:

```
[PM2] Init System found: systemd
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

Run the provided command (ensure you adjust the username and home path if necessary):

```sh
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

Your Node.js/Express.js application is now set up to start on boot with PM2.

---

## Configuring Nginx as a Reverse Proxy

Nginx can be used to serve your Node.js application by acting as a reverse proxy. This setup can help with load balancing, SSL termination, and serving static files.

### 1. Install Nginx

```sh
sudo apt install nginx
```

### 2. Configure Nginx

There are two common methods for configuring Nginx:

---

### **Method 1: Using `sites-available`/`sites-enabled`**

1. **Create a New Configuration File**
    
    Create a new file in `/etc/nginx/sites-available/` (e.g., `node_app`):
    
    ```sh
    sudo nano /etc/nginx/sites-available/node_app
    ```
    
2. **Add the Following Configuration**
    
    Adjust the values as needed:
    
    ```nginx
    server {
        listen 80;
        server_name your_domain_or_IP;
    
        location / {
            proxy_pass http://127.0.0.1:3000;  # Replace 3000 with your Node.js application's port
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
    ```
    
3. **Enable the Configuration**
    
    Create a symbolic link to enable your new configuration:
    
    ```sh
    sudo ln -s /etc/nginx/sites-available/node_app /etc/nginx/sites-enabled/
    ```
    

---

### **Method 2: Using the `conf.d` Directory**

4. **Create a Configuration File**
    
    Create a new file in `/etc/nginx/conf.d/` (e.g., `node_app.conf`):
    
    ```sh
    sudo nano /etc/nginx/conf.d/node_app.conf
    ```
    
5. **Add the Following Configuration**
    
    Adjust the values as needed:
    
    ```nginx
    server {
        listen 80;
        server_name your_domain_or_IP;
    
        location / {
            proxy_pass http://127.0.0.1:3000;  # Replace 3000 with your Node.js application's port
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
    ```
    

_Note:_ Using `conf.d` is a simpler alternative where each configuration file is automatically included by Nginx. You typically don’t need to create symbolic links.

---

### 3. Test and Restart Nginx

Test the configuration for syntax errors:

```sh
sudo nginx -t
```

If the test is successful, restart Nginx:

```sh
sudo systemctl restart nginx
```

Your Node.js application should now be accessible through Nginx on port 80.

---

**DONE**
