# CraveCorner — Linux-Hosted Food Website

CraveCorner is a beginner-friendly, static food-menu website created to practise Linux fundamentals and basic website hosting. It has a dark theme, orange accents, a hero section, food cards, navigation links, and responsive styling.

The goal of this project is to learn how to create website files in Linux, serve them locally, manage the server process, and (optionally) run the website as a `systemd` service.

> **Project status:** The HTML/CSS website and manual Python HTTP-server workflow are the core project. The `systemd` service is an optional next step and should be marked complete only after you create it and verify that it works.

## Table of Contents

- [Features](#features)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [1. Create the Project Directory](#1-create-the-project-directory)
- [2. Create the HTML and CSS Files](#2-create-the-html-and-css-files)
- [3. Run the Website Locally](#3-run-the-website-locally)
- [4. Check the Website from the Terminal](#4-check-the-website-from-the-terminal)
- [5. Run the Website as a systemd Service](#5-run-the-website-as-a-systemd-service)
- [6. Useful Service Commands](#6-useful-service-commands)
- [7. Troubleshooting](#7-troubleshooting)
- [Linux Commands Learned](#linux-commands-learned)
- [Limitations and Security Notes](#limitations-and-security-notes)
- [Future Improvements](#future-improvements)
- [Upload the Project to GitHub](#upload-the-project-to-github)

## Features

- Responsive food-menu landing page.
- Navigation links to Home, Our Menu, and About.
- Food cards showing sample dishes and prices.
- Dark background with orange highlights.
- CSS hover effects and layouts for desktop and smaller screens.
- Local hosting through Python 3's built-in HTTP server.
- Optional Linux service management using `systemd`.

## Technology Stack

| Technology | Purpose |
|---|---|
| Linux / Ubuntu / WSL | Operating environment and command-line practice |
| HTML5 | Structure and content of the website |
| CSS3 | Colours, spacing, layout, hover effects, and responsive design |
| Python 3 `http.server` | Serves the static files over HTTP for local testing |
| `systemd` (optional) | Starts, stops, restarts, and monitors the server as a Linux service |
| Git and GitHub | Version control and project sharing |

## Project Structure

After setting up the files, the directory looks like this:

```text
cravecorner/
├── index.html
├── style.css
└── README.md
```

If you configure the optional service, its unit file lives outside the project directory:

```text
/etc/systemd/system/cravecorner.service
```

- `index.html` contains the webpage content and links to the stylesheet.
- `style.css` contains the page styling and responsive rules.
- `README.md` documents the project, setup, commands, and troubleshooting.
- `cravecorner.service` is a system-level configuration file used by `systemd`; it is not stored in the website directory by default.

## Prerequisites

- Ubuntu, another Linux distribution, or Ubuntu under WSL.
- Python 3 installed.
- A terminal and text editor such as `nano`.
- A web browser.
- `systemd` running if you want to use the service-management section.

Check the tools:

```bash
whoami
pwd
python3 --version
command -v python3
```

What these commands do:

- `whoami` prints the current Linux username.
- `pwd` prints the current working directory.
- `python3 --version` checks whether Python 3 is installed.
- `command -v python3` prints the path to the Python executable. Use that path in the service file if it differs from `/usr/bin/python3`.

## 1. Create the Project Directory

Create the project folder under your home directory:

```bash
mkdir -p ~/projects/cravecorner
```

- `mkdir` creates a directory.
- `-p` creates any missing parent directories and does not complain if the directory already exists.
- `~` represents your home directory.

Move into the folder and verify the path:

```bash
cd ~/projects/cravecorner
pwd
```

Create the website files:

```bash
touch index.html style.css
ls -l
```

- `cd` changes the current directory.
- `touch` creates empty files if they do not exist.
- `ls -l` lists files in a detailed format, including permissions, owner, size, and modification time.

## 2. Create the HTML and CSS Files

### Create `index.html`

```bash
nano index.html
```

Paste the HTML code for the CraveCorner page into the editor.

Save and exit Nano:

1. Press `Ctrl+O` to write the file.
2. Press `Enter` to confirm the filename.
3. Press `Ctrl+X` to exit.

### Create `style.css`

```bash
nano style.css
```

Paste the CSS styling into the editor and save using `Ctrl+O`, `Enter`, and `Ctrl+X`.

### Verify the files

```bash
ls -l
head -n 20 index.html
head -n 20 style.css
```

- `head -n 20` displays the first 20 lines so you can confirm that the files contain code.
- `cat index.html` can display the entire HTML file in the terminal, although long files are often easier to inspect with `less index.html`.

The HTML file should link to the stylesheet with:

```html
<link rel="stylesheet" href="style.css">
```

The filename and capitalization must match the actual CSS filename.

## 3. Run the Website Locally

Move to the project directory:

```bash
cd ~/projects/cravecorner
```

Start Python's built-in HTTP server:

```bash
python3 -m http.server 8000
```

This command:

- Runs Python 3 as a module.
- Starts a basic HTTP server.
- Uses port `8000`.
- Serves files from the current directory.
- Uses `index.html` as the default page when present.

Open the following address in your browser:

```text
http://localhost:8000
```

To stop the server, return to the terminal and press `Ctrl+C`.

### Bind address: local-only versus network listening

For a local-only server, use:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

To listen on all network interfaces, use:

```bash
python3 -m http.server 8000 --bind 0.0.0.0
```

`0.0.0.0` is a bind address, not an address to type into the browser. Listening on all interfaces can make the site reachable by other devices when the network and firewall allow it. For basic local practice, prefer `127.0.0.1`. Do not expose Python's development server directly to the public internet.

## 4. Check the Website from the Terminal

With the server running, open a second terminal.

### Check the HTTP response

```bash
curl -I http://localhost:8000
```

`curl` sends a request to the web server. `-I` requests response headers only. A response such as `HTTP/1.0 200 OK` indicates that the server responded successfully.

### Check the server process

```bash
ps aux | grep '[h]ttp.server'
```

- `ps aux` lists running processes.
- `grep` filters output to matching text.
- The `[h]` pattern helps avoid matching the `grep` command itself.

### Check listening TCP ports

```bash
ss -ltn
```

- `ss` shows socket information.
- `-l` filters to listening sockets.
- `-t` shows TCP sockets.
- `-n` shows numeric ports rather than resolving service names.

To look specifically for port 8000:

```bash
ss -ltn | grep ':8000'
```

## 5. Run the Website as a systemd Service

This is an optional part of the project. It lets you manage the website through `systemctl` instead of starting the Python server manually.

### Step 5.1 — Check systemd

```bash
systemctl is-system-running
```

If systemd is not running, service commands may fail. On WSL, systemd must be enabled and supported in the distribution before continuing. Fix that first rather than assuming the service was created successfully.

### Step 5.2 — Confirm your username and Python path

```bash
whoami
command -v python3
```

For the example unit below, replace `YOUR_USERNAME` with the username printed by `whoami`. Replace `/usr/bin/python3` if `command -v python3` prints a different path.

### Step 5.3 — Create the unit file

```bash
sudo nano /etc/systemd/system/cravecorner.service
```

Paste the following configuration, replacing every instance of `YOUR_USERNAME`:

```ini
[Unit]
Description=CraveCorner Food Website
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/home/YOUR_USERNAME/projects/cravecorner
ExecStart=/usr/bin/python3 -m http.server 8000 --bind 127.0.0.1
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Save with `Ctrl+O`, `Enter`, and `Ctrl+X`.

**What the settings mean:**

| Setting | Meaning |
|---|---|
| `[Unit]` | General information and startup ordering |
| `Description` | Human-readable service description |
| `After=network.target` | Orders startup after the network target |
| `[Service]` | Defines how the process runs |
| `Type=simple` | Treats the process started by `ExecStart` as the main service process |
| `User` | Runs the server as the specified unprivileged Linux user |
| `WorkingDirectory` | Sets the directory from which Python serves files |
| `ExecStart` | Command used to start the server |
| `Restart=on-failure` | Restarts the service after an unexpected failure |
| `[Install]` | Describes how the service is enabled |
| `WantedBy=multi-user.target` | Allows enabling it for the normal multi-user startup target |

The example binds to `127.0.0.1`, so the site is intended for local access only. If you deliberately need same-network access later, learn the implications of binding to `0.0.0.0` and configure the firewall accordingly.

### Step 5.4 — Reload service definitions

```bash
sudo systemctl daemon-reload
```

This tells systemd to reload its unit-file definitions after a new or changed service file.

### Step 5.5 — Enable the service at boot

```bash
sudo systemctl enable cravecorner
```

`enable` configures the service to start automatically at boot. It does not, by itself, start the service immediately.

### Step 5.6 — Start the service now

```bash
sudo systemctl start cravecorner
```

### Step 5.7 — Verify it

```bash
sudo systemctl status cravecorner
curl -I http://localhost:8000
```

Look for `active (running)` in the service status and an HTTP response in the `curl` output. If the status says `failed`, inspect the service logs in the troubleshooting section.

## 6. Useful Service Commands

| Task | Command | What it does |
|---|---|---|
| Start | `sudo systemctl start cravecorner` | Starts the service now |
| Stop | `sudo systemctl stop cravecorner` | Stops the running service |
| Restart | `sudo systemctl restart cravecorner` | Restarts the service |
| Status | `sudo systemctl status cravecorner` | Shows current state and recent status information |
| Enable at boot | `sudo systemctl enable cravecorner` | Configures automatic startup |
| Disable at boot | `sudo systemctl disable cravecorner` | Removes automatic-start configuration |
| View recent logs | `sudo journalctl -u cravecorner -n 50 --no-pager` | Displays the latest 50 log lines |
| Follow logs live | `sudo journalctl -u cravecorner -f` | Streams new log entries; press `Ctrl+C` to exit |
| Reload unit files | `sudo systemctl daemon-reload` | Reloads service definitions after unit-file changes |

Important distinction:

- `start` and `stop` control the service right now.
- `enable` and `disable` control whether it is configured to start automatically at boot.
- If you edit the `.service` file, run `sudo systemctl daemon-reload` before restarting the service.

## 7. Troubleshooting

### Error: `Address already in use`

Something is already listening on port 8000. Check:

```bash
ss -ltnp | grep ':8000'
```

If the earlier manual server is running in a terminal, switch to that terminal and press `Ctrl+C`. Then start the service again:

```bash
sudo systemctl restart cravecorner
```

### Service fails to start

Check the status and logs:

```bash
sudo systemctl status cravecorner
sudo journalctl -u cravecorner -n 50 --no-pager
```

Common causes include an incorrect username, an incorrect project path, an incorrect Python executable path, or port 8000 already being occupied.

### Website files return a 404 error

Check that the service is serving the correct directory:

```bash
ls -l ~/projects/cravecorner
```

Confirm that `index.html` and `style.css` exist and that the HTML refers to `style.css` with the correct spelling.

### `systemctl` reports that systemd is unavailable

Check:

```bash
systemctl is-system-running
```

On WSL, verify that systemd is enabled for the Ubuntu distribution. The Python web server can still be tested manually even if systemd is not yet available.

### CSS looks missing

Confirm that the stylesheet exists and the HTML link is correct. Also note that the website imports Google Fonts and uses externally hosted food images, so those items need an internet connection. Basic local HTML and CSS files are still served from your project directory.

## Linux Commands Learned

| Command | How it is used in this project |
|---|---|
| `whoami` | Identify the current Linux user |
| `pwd` | Print the current directory |
| `mkdir -p` | Create the project directory and parent directories |
| `cd` | Move into the project directory |
| `touch` | Create the HTML and CSS files |
| `ls -l` / `ls -la` | Inspect files, details, and hidden files |
| `nano` | Edit HTML, CSS, and service configuration files |
| `head` / `cat` / `less` | Inspect file contents |
| `python3 --version` | Check the installed Python version |
| `python3 -m http.server` | Serve the static website locally |
| `curl -I` | Check an HTTP response |
| `ps aux` | Inspect running processes |
| `grep` | Filter output for relevant text |
| `ss -ltn` | Inspect listening TCP ports |
| `chmod +x` | Make a script executable, if a shell script is added |
| `systemctl` | Start, stop, restart, enable, and inspect services |
| `journalctl` | Read systemd service logs |
| `sudo` | Run administrative commands with elevated privileges |

## Limitations and Security Notes

- This is currently a **static website**, not a complete food-ordering web application.
- The displayed dishes and prices are sample content.
- It does not have a working cart, order submission, customer accounts, database, payment processing, or order tracking.
- Python's built-in `http.server` is intended for simple testing, not production hosting.
- Binding to `0.0.0.0` makes the server listen on all network interfaces; whether other devices can connect also depends on networking and firewall rules. Avoid exposing this basic server to the public internet.
- External images and fonts need internet access.
- A working `systemd` service depends on systemd being available and running on the Linux environment.

## Future Improvements

1. Add JavaScript interactions and a basic cart.
2. Build a Python Flask backend for handling order submissions.
3. Store test orders in an SQLite database.
4. Add automated tests and structured application logs.
5. Create a Docker image and document container commands.
6. Build a GitHub Actions workflow for automated checks.
7. Deploy securely to a remote host with a production-appropriate web server and HTTPS.

These are future tasks, not current features.

## Upload the Project to GitHub

If the GitHub repository has already been created, run the following from the project directory. Replace the example repository URL with your own repository URL.

```bash
cd ~/projects/cravecorner
git init
git branch -M main
git add index.html style.css README.md
git status
git commit -m "Add CraveCorner Linux-hosted website"
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY.git
git push -u origin main
```

What each Git command does:

- `git init` creates a local Git repository.
- `git branch -M main` names the current branch `main`.
- `git add` stages the selected files for the next commit.
- `git status` shows staged and unstaged changes.
- `git commit` records the staged changes in local Git history.
- `git remote add origin` connects the local repository to the GitHub repository.
- `git push -u origin main` uploads the commit and sets the upstream branch.

If `origin` is already configured, inspect it with `git remote -v` rather than adding it again. GitHub authentication may be required when pushing.

---

## Learning Outcome

By completing this project, you practise creating website files, navigating the Linux filesystem, running a local HTTP server, checking processes and network ports, reading logs, and optionally managing a long-running process with `systemd`.

**Author:** Kanishka Bhardwaj  
**Project:** CraveCorner — Linux-hosted food-menu website  
**Purpose:** Beginner Linux practice and gradual progression toward system administration and DevOps
