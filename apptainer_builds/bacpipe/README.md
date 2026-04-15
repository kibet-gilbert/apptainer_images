What BacPipe’s Docker command is doing can be mapped almost one-to-one to **Apptainer/Singularity**, but with some important conceptual differences (especially networking and X11).

Below I’ll break it down **option by option**, then give you a **working Apptainer equivalent** that is appropriate for HPC / shared Unix systems.

---

## 1. Key differences: Docker vs Apptainer (important)

| Docker                             | Apptainer                                    |
| ---------------------------------- | -------------------------------------------- |
| Containers are isolated by default | Containers run **as you**, using host kernel |
| `--net=host` needed                | **Host networking is default**               |
| Root inside container              | **No root** (security feature)               |
| Needs `-v` for many system paths   | `$HOME`, `/tmp`, `/proc`, `/dev` auto-bound  |
| GUI/X11 needs explicit wiring      | X11 works via bind + env                     |

👉 **Result:** Apptainer commands are shorter and safer.

---

## 2. Original Docker command (for reference)

```bash
docker run -it \
  --hostname "YOUR_HOST_ID" \
  --net=host \
  -e DISPLAY=${DISPLAY} \
  -v ${HOME}/.Xauthority:/root/.Xauthority \
  -v <local_data_folder>:<container_data_folder> \
  mahmed/bacpipe python ./Pipeline.py unix
```

---

## 3. Apptainer equivalent (recommended)

### Step 1: Pull image (already done)

```bash
apptainer pull bacpipe_latest.sif docker://mahmed/bacpipe:latest
```

---

### Step 2: Enable X11 forwarding (host side)

If you are **SSH-ing into a server**:

```bash
ssh -X gkibet@server
```

(or `-Y` if needed)

Verify:

```bash
echo $DISPLAY
```

---

### Step 3: Run BacPipe with Apptainer

```bash
apptainer exec \
  --bind /path/to/local_data:/data \
  --env DISPLAY=$DISPLAY \
  bacpipe_latest.sif \
  python ./Pipeline.py unix
```

✔ This already replaces:

* `--net=host` → **default in Apptainer**
* `-e DISPLAY=...`
* `-v local_data:container_data`

---

## 4. X11 support (important details)

### `.Xauthority`

Unlike Docker, Apptainer **automatically binds your `$HOME`**, so this is usually unnecessary.

If BacPipe **explicitly requires** it:

```bash
apptainer exec \
  --bind $HOME/.Xauthority:$HOME/.Xauthority \
  --env DISPLAY=$DISPLAY \
  bacpipe_latest.sif \
  python ./Pipeline.py unix
```

👉 **Do NOT bind to `/root/.Xauthority`**
Apptainer does not run as root.

---

## 5. Hostname handling

Docker:

```bash
--hostname "YOUR_HOST_ID"
```

Apptainer:

* You **cannot spoof hostname by default** (security reason)
* BacPipe rarely needs this
* If it truly matters, you can pass it as an env variable:

```bash
apptainer exec \
  --env HOSTNAME=$(hostname) \
  bacpipe_latest.sif \
  python ./Pipeline.py unix
```

---

## 6. Interactive shell (debugging)

Equivalent of `docker run -it`:

```bash
apptainer shell bacpipe_latest.sif
```

Then inside:

```bash
python ./Pipeline.py unix
```

---

## 7. Full “closest possible” equivalent command

```bash
apptainer exec \
  --bind /path/to/local_data:/data \
  --bind $HOME/.Xauthority:$HOME/.Xauthority \
  --env DISPLAY=$DISPLAY \
  bacpipe_latest.sif \
  python ./Pipeline.py unix
```

---

## 8. Common HPC-specific caveats (very relevant for you)

### ❌ GUI on compute nodes

* Most clusters **do not allow X11 on compute nodes**
* Run GUI steps on:

  * login node
  * interactive job (`srun --x11`)
  * local workstation

### ✔ Batch mode recommended

If BacPipe supports non-GUI execution:

```bash
python ./Pipeline.py unix --nogui
```

(or equivalent flag)

---

## 9. Summary mapping table

| Docker              | Apptainer                  |
| ------------------- | -------------------------- |
| `docker run -it`    | `apptainer exec` / `shell` |
| `--net=host`        | default                    |
| `-v a:b`            | `--bind a:b`               |
| `-e VAR=val`        | `--env VAR=val`            |
| `/root/.Xauthority` | `$HOME/.Xauthority`        |
| root container      | user container             |

---
