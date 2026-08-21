[فارسی](README_FA.md)

# Free Cloud Desktop

Temporary Windows and Ubuntu machines running through GitHub Actions, with remote access over Tailscale.

Three workflows are included:

| # | Environment | Access | Workflow |
|---|---|---|---|
| 1 | Windows | Desktop (RDP) | `windows.yml` |
| 2 | Ubuntu Desktop | XFCE desktop (RDP) | `ubuntu_desktop.yml` |
| 3 | Ubuntu | Command line (SSH) | `ubuntu_ssh.yml` |

All three workflows use the same two GitHub Secrets. Depending on the workflow, common tools such as FFmpeg, Python, Pillow, ImageMagick, and Firefox are installed automatically.

---

## Setup

The setup is the same for all three workflows.

### 1. Create the GitHub Secrets

Go to:

**Repository → Settings → Secrets and variables → Actions → New repository secret**

Create these two Secrets:

| Secret | Value |
|---|---|
| `RDP_PASSWORD` | A strong password for the `PersianPl` user |
| `TAILSCALE_AUTHKEY` | A Tailscale auth key |

### Generate a 32-character password

**WSL / Linux:**

```bash
tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 32; echo
```

**Windows CMD (using PowerShell):**

```cmd
powershell -Command "-join ((48..57)+(65..90)+(97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})"
```

### Create the Tailscale Auth Key

Open the Tailscale admin console and create an auth key.

Enable:

- **Reusable** — so the key can be used for multiple workflow runs.
- **Ephemeral** — so temporary machines are removed automatically.

### 2. File structure

```text
.github/workflows/
├── windows.yml
├── ubuntu_desktop.yml
└── ubuntu_ssh.yml

README.md
README_FA.md
```

### 3. Run a workflow

Go to:

**Actions → select a workflow → Run workflow**

After the workflow starts the machine, the Tailscale IP address is printed in the workflow logs.

---

## Windows RDP

A full Windows graphical desktop.

- **Startup:** about 1 minute
- **Tools:** FFmpeg + Python + Pillow
- **Connection:** `mstsc` on Windows or Remmina on Linux

| Field | Value |
|---|---|
| Address | Tailscale IP |
| Username | `PersianPl` |
| Password | Value of `RDP_PASSWORD` |

---

## Ubuntu Desktop

A lightweight XFCE desktop with xrdp.

- **Startup:** about 3–5 minutes
- **Tools:** FFmpeg + Python + Pillow + ImageMagick + Firefox
- **Connection:** any RDP client such as mstsc, Remmina, or Microsoft Remote Desktop

| Field | Value |
|---|---|
| Address | Tailscale IP |
| Username | `PersianPl` |
| Password | Value of `RDP_PASSWORD` |

> If you get a black screen after connecting, wait a few seconds or reconnect while the XFCE session finishes loading.

---

## Ubuntu SSH

The lightest and fastest option. No graphical desktop is installed.

- **Startup:** about 1 minute
- **Tools:** FFmpeg + Python + Pillow + ImageMagick
- **Connection:**

```bash
ssh PersianPl@<Tailscale-IP>
```

Password:

```text
RDP_PASSWORD
```

The workflow also enables Tailscale SSH with `--ssh`. If your device is on the same Tailscale network, Tailscale SSH can also be used.

---

## Which one should I use?

| If you want to... | Use |
|---|---|
| Run scripts, download files, or process data | Ubuntu SSH |
| Use command-line content tools | Ubuntu SSH |
| Work with a Linux graphical desktop | Ubuntu Desktop |
| Run Windows-only software | Windows RDP |

---

## Important notes

- **Temporary machines:** Each run stays alive for up to 6 hours. The machine and its data are removed afterwards. Do not keep anything important on the machine.
- **Changing public IP:** The public internet IP can change between runs, so this setup is not intended for services that require a fixed IP.
- **Best suited for:** Short tasks, temporary processing or downloads, and content creation.
- **GitHub Actions:** Use GitHub Actions resources responsibly and within GitHub's terms.

---

## Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| `invalid key: unable to validate API key` | Tailscale key is invalid, expired, or incomplete | Create a new auth key and copy the full value into `TAILSCALE_AUTHKEY` |
| User creation step fails | `RDP_PASSWORD` is empty or does not meet the password requirements | Check the `RDP_PASSWORD` Secret |
| RDP/SSH connection does not work | Tailscale is not running on the client | Install Tailscale on the client and sign in |
| Black screen on Ubuntu Desktop | XFCE is still starting | Wait a few seconds or reconnect |
| `Permission denied` during SSH | Incorrect password | Check the `RDP_PASSWORD` Secret |
