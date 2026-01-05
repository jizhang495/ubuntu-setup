# Ubuntu Scientific Workstation Setup

This guide targets an experimental scientist who needs an Ubuntu workstation for research, paper drafting, data analysis, figure generation, instrument control, and web app prototyping. Work through the sections in order; each builds on the previous one. Keyboard shortcuts to open a terminal: `Ctrl`+`Alt`+`T` inside GNOME, or switch to a virtual console with `Ctrl`+`Alt`+`F3/F2` if the desktop is unavailable.

## Companion guides

- [Git setup and workflow](git-guide.md)
- [uv setup and guidelines](uv-guide.md)
- [Docker guide](docker-guide.md)
- [Google Cloud (gcloud) guide](gcloud-guide.md)

## 1. Connect to eduroam (Cambridge)

The UIS script from <https://help.uis.cam.ac.uk/service/wifi/connect-to-eduroam/linux> configures NetworkManager for eduroam.

Download `eduroam-linux-UoC-eduroam_public_CA.py` from <cat.eduroam.org>

```bash
sudo apt update
sudo apt install -y python3.12-venv python-is-python3 python3-dbus python3-gi network-manager
cd ~/Downloads
python eduroam-linux-UoC-eduroam_public_CA.py
```

Reboot or restart NetworkManager (`sudo systemctl restart NetworkManager`) and verify that `nmcli connection show` reports an `eduroam` profile. Keep the `.py` script for future certificate renewals.

## 2. System updates & base packages

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y build-essential curl wget git ca-certificates unzip zip htop neovim software-properties-common gnome-tweaks
```

These packages cover compilation, shell basics, editors, and desktop tweaks. Add any institution-specific CA certificates with `sudo update-ca-certificates` if your lab proxies HTTPS traffic.

## 3. Python environments

```bash
sudo apt install -y python3-pip python3-dev python3-venv pipx
pipx ensurepath   # log out/in or source ~/.profile afterwards
python -m venv ~/venvs/default
source ~/venvs/default/bin/activate
python -m pip install --upgrade pip
```

Use `pipx install <tool>` for global command-line utilities (e.g., `pipx install jupyterlab black pre-commit`). Inside project-specific virtual environments install the scientific stack:

```bash
pip install numpy scipy pandas scikit-learn matplotlib seaborn plotly jupyterlab
pip install xarray pint uncertainties pyvisa pyvisa-py pyusb
```

For reproducibility, capture versions in `requirements.txt` or, for heavier projects, consider `mamba`/`conda` if you rely on compiled libraries or GPU work.

## 4. Editors, IDEs, and PDF tools

Install Visual Studio Code from Microsoft’s repository so updates arrive via APT. Follow <https://code.visualstudio.com/docs/setup/linux>.

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /usr/share/keyrings/ms.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/ms.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
sudo apt update
sudo apt install -y code
```

Suggested extensions: Python, Jupyter, Markdown All in One, GitHub Pull Requests, Docker, and PDF Viewer (or rely on GNOME’s `evince`). For collaborative writing consider the `code --install-extension ms-toolsai.jupyter` flow or link VS Code with Overleaf via Git.

## 5. Containers and reproducible environments

```bash
# Install Docker Engine (Docker CE) on Ubuntu (per docs.docker.com)
# Remove potentially-conflicting packages (safe if not installed)
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt remove -y $pkg
done

sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

Use Docker for pinned analysis environments, instrument-control services, or web back ends. Add Compose files per project (e.g., `docker compose up -d`). See `docker-guide.md` for common commands and workflows.

## 6. Node.js and web tooling

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

Install `npm install --global yarn pnpm` if your web apps need those package managers. Keep an LTS version for stability; use `nvm` when you need to test multiple Node versions.

## 7. Cloud & Git integration

```bash
# Google Cloud SDK
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt update
sudo apt install -y google-cloud-cli

# GitHub CLI
sudo apt install -y gh
gh auth login
```

Configure Git identities (`git config --global user.name "Your Name"`). If collaborating across GitHub and institutional GitLab, install the respective CLIs for easy issue tracking and PR workflows.

## 8. Data analysis, statistics, and notebooks

- **R & statistics:** `sudo apt install -y r-base r-base-dev` plus `install.packages(c("tidyverse","sf","shiny"))` inside R.
- **Jupyter:** `pipx install jupyterlab` for a workspace-wide server, then add kernels (`python -m ipykernel install --user --name lab-default`).
- **Databases:** install `sudo apt install -y sqlite3 postgresql-client mysql-client` to interact with lab data sources.
- **Plotting/graphics:** the Python packages from section 3 plus domain-specific ones (e.g., `pip install bokeh holoviews plotly-express`).

Document your preferred stack in a lab README so students and collaborators can recreate it quickly.

## 9. Document preparation and reference management

```bash
sudo apt install -y texlive-full pandoc ghostscript graphviz
```

Use Zotero 7 for citation management:

```bash
wget https://download.zotero.org/client/release/7.0.0/Zotero-7.0.0_linux-x86_64.tar.bz2
tar -xjf Zotero-*.tar.bz2
sudo mv Zotero_linux-x86_64 /opt/zotero
sudo ln -s /opt/zotero/zotero.desktop /usr/share/applications/zotero.desktop
```

Add the Zotero connector to your browser, enable BetterBibTeX for citekey control, and synchronize your library with Zotero’s cloud account.

## 10. Visualization & 3D content

```bash
sudo apt install -y inkscape blender gimp krita
```

Use Inkscape for SVG figures and Blender for 3D apparatus renders. Keep color profiles consistent (e.g., export to CMYK when preparing journal-ready figures). Link VS Code or Git to your figure repository to track revisions.

## 11. Storage, syncing, and secrets

OneDriver: file on demand, mounts as a virtual drive (recommended)

```bash
sudo add-apt-repository ppa:jstaf/onedriver
sudo apt update
sudo apt install onedriver
```

OneDrive (by abraunegg): Full Sync, downloads everything locally.

```bash
sudo apt install -y onedrive
onedrive --synchronize
```

For secure credential storage use Bitwarden (`sudo snap install bitwarden` or the Firefox/Chrome extension). If you remain on NordPass, note that only the browser extension or CLI is available on Linux; install the CLI via `curl https://downloads.npass.app/linux/app/nordpass_latest_amd64.deb -o nordpass.deb && sudo apt install ./nordpass.deb`.

## 12. Instrument control & hardware interfaces

- USB/serial tools: `sudo apt install -y python3-usb python3-serial usbutils minicom`.
- VISA/GPIB: `pip install pyvisa pyvisa-py`, add NI drivers if your hardware requires them.
- Data acquisition: `sudo apt install -y libfftw3-dev libhdf5-dev` for FFTs and large file formats.
- Real-time plotting or dashboards: `pip install dash streamlit panel`.

Keep udev rules (e.g., `/etc/udev/rules.d/99-lab-instruments.rules`) under version control to ensure instruments appear with the correct permissions.

## 13. Optional desktop polish

- Install fonts (`sudo apt install -y fonts-firacode fonts-dejavu fonts-inter`). For Georgia install Microsoft’s core fonts (`sudo apt install -y ttf-mscorefonts-installer`) and accept the license prompt to match collaborators’ Word documents.
- Enable GNOME extensions (dynamic workspaces, clipboard manager) via `sudo apt install -y gnome-shell-extensions`.
- Configure backups with `deja-dup` pointing to institutional storage.

## 14. Validation checklist

1. Run `sudo apt update && sudo apt upgrade -y` without errors.
2. Connect to eduroam and confirm `ping www.cam.ac.uk` succeeds.
3. Launch VS Code, sign into GitHub, and sync settings.
4. Create and activate a Python virtual environment, run `python -c "import numpy, pandas, matplotlib"`.
5. Launch JupyterLab and open a notebook from your research repo.
6. `docker run hello-world` completes and `gcloud auth login` succeeds.
7. Onedrive sync completes without authentication prompts.

Document any deviations or lab-specific scripts in this repository so the next setup only needs light adjustments.
