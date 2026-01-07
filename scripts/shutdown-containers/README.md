# Shutdown All Docker Containers

*(Optional)* Add container names to the `shutdown_last` array to shut them down after all other containers.
Example: `shutdown_last=("postgresql" "mariadb")`.

```bash
#!/bin/bash

shutdown_last=()

# Function to check if a container is in the `shutdown_last` array
is_shutdown_last() {
    local name="$1"
    for last in "${shutdown_last[@]}"; do
        if [[ "$name" == "$last" ]]; then
            return 0
        fi
    done
    return 1
}

# Get all running container names
mapfile -t containers < <(docker ps --format '{{.Names}}')

# Shutdown all containers except those in `shutdown_last`
echo -e "\nShutting down containers..."
for name in "${containers[@]}"; do
    if ! is_shutdown_last "$name"; then
        echo "  Stopping: $name"
        docker stop "$name"
    fi
done

# Shutdown containers in the `shutdown_last` array
if [ ${#shutdown_last[@]} -gt 0 ]; then
    echo -e "\nShutting down last containers..."
    
    for name in "${shutdown_last[@]}"; do
        if docker ps --format '{{.Names}}' | grep -q "^${name}$"; then
            echo "  Stopping: $name"
            docker stop "$name"
        fi
    done
fi

echo -e "\nDone! All containers have been shut down."
```
