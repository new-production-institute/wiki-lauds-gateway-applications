## Build and launch Docker containers

Most Docker containers are off-the-shelf, but the Node-RED container is built with some useful plugins included. To build and run these Docker containers in a single step:

Assuming required credentials above are set in `.env`

```sh
source .env && docker compose --file software/container/docker-compose.yml up --force-recreate --build
```


## Docker Volume Backup

### Create Backup Directory

```bash
mkdir -p ~/docker-backups
cd ~/docker-backups
```

Create two files:

- `backup_volumes.sh`
- `restore_volumes.sh`

---

### Backup Script

```bash
#!/bin/bash
# Backup Docker volumes
# Usage: ./backup_volumes.sh

BACKUP_DIR=backups_$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# List of volumes to back up
VOLUMES=(
  container_grafana-dashboards
  container_grafana-data
  container_influxdb2
  container_jupyter-notebook-data
  container_nodered-data
  container_server-data
)

echo "Starting backup of Docker volumes..."

for VOLUME in "${VOLUMES[@]}"; do
  echo "Backing up $VOLUME..."
  docker run --rm \
    -v ${VOLUME}:/volume \
    -v $(pwd)/${BACKUP_DIR}:/backup \
    alpine sh -c "cd /volume && tar -czf /backup/${VOLUME}.tar.gz ."
done

echo "Backup complete!"
echo "Backups saved in: $(pwd)/${BACKUP_DIR}"
```

---

### Restore Script

```bash
#!/bin/bash
# Restore Docker volumes from backup folder
# Usage: ./restore_volumes.sh backups_20251105_142210

BACKUP_DIR=$1

if [ -z "$BACKUP_DIR" ]; then
  echo "Please specify the backup directory (e.g., ./restore_volumes.sh backups_20251105_142210)"
  exit 1
fi

if [ ! -d "$BACKUP_DIR" ]; then
  echo "Backup directory not found: $BACKUP_DIR"
  exit 1
fi

echo "Starting restore from $BACKUP_DIR..."

for FILE in ${BACKUP_DIR}/*.tar.gz; do
  VOLUME=$(basename ${FILE%.tar.gz})
  echo "Restoring $VOLUME..."

  docker volume create $VOLUME >/dev/null 2>&1

  docker run --rm \
    -v ${VOLUME}:/volume \
    -v $(pwd)/${BACKUP_DIR}:/backup \
    alpine sh -c "cd /volume && tar -xzf /backup/${VOLUME}.tar.gz"
done

echo "Restore complete!"
```

---

### Make Scripts Executable

```bash
chmod +x backup_volumes.sh
chmod +x restore_volumes.sh
```

---

### Usage

#### Backup Docker Volumes

```bash
./backup_volumes.sh
```

This creates a folder similar to:

```
backups_20251105_142210
```

---

#### Restore Docker Volumes

```bash
./restore_volumes.sh backups_20251105_142210
```